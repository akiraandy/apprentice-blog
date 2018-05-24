---
layout: post
title: "AWS and Docker"
description: "Where highly configurable scalability meets containerization."
categories: [code]
tags: [aws docker]
redirect_from:
  - /2018/05/23
---
These past two weeks have been a crash course into the world of AWS and Docker. My task was to Dockerize a Rails app and then deploy it to Elastic Beanstalk and hook it up to an RDS instance. Whew. What does that mean? Let's first explore what Docker is.

If you've done any sort of development, chances are high that you've heard of Docker. I had heard about it a lot but never really knew the what/how/why of the service. Essentially it's a really handy way of making your application portable. In the time before Docker, you could build an application with all it's dependencies declared and it would run fine on your local machine. But let's say your friend wants to run your application on their machine. Of course it breaks! Their system has different versions of your app's dependencies installed, not to mention a whole bunch of other finicky configuration and environment variables out of sync with your setup. What a mess! But fret not, Docker is here to save the day!

Docker isolates software into a container. A container virtualizes the operating system that an application runs in while packaging that app's source code and dependencies together. Docker is an app that is designed to communicate to your OS and abstract all the nitty gritty details of your app's runtime environment away from anyone who wants to use your Dockerized app. As long as your Dockerized application runs in it's container, it can run anywhere Docker can be run. This makes it a very powerful tool for ensuring reliability, security and portability of your software.

With that being said, how do you setup your application to be Dockerized? Let me walk you through how I ended up Dockerizing my Rails application.
```
FROM ruby:2.5.0
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
ENV RAILS_ENV production
ENV RAILS_SERVE_STATIC_FILES true
ENV RAILS_LOG_TO_STDOUT true
ENV SECRET_KEY_BASE ***
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install --without development test
COPY . /myapp
RUN bundle exec rake assets:precompile
EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
```
Let's go through this. First we have to make a ```Dockerfile``` which defines what goes on in the environment of our container.
Then inside of this file we have to define the official runtime of our application, which in our case is Ruby 2.5.0
```
FROM ruby:2.5.0
```

The next line is installing some basic Linux packages/dependencies including nodejs
```
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
```

A lot of these commands might be familiar to you because they're basic Linux terminal commands!
Here we are creating a directory called ```myapp``` which will be our working directory which we define in the second command.
```
RUN mkdir /myapp
WORKDIR /myapp
```

The following commands are setting environment variables within the container (these are not necessary if you are trying to run your app in a testing or development environment)
```
ENV RAILS_ENV production
ENV RAILS_SERVE_STATIC_FILES true
ENV RAILS_LOG_TO_STDOUT true
ENV SECRET_KEY_BASE ***
```

From here we are copying our ```Gemfile``` and ```Gemfile.lock``` int our working directory from our current directory.
```
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
```

Now we install all our dependencies via ```bundle install``` like you would normally when pulling down a Ruby application.
```
RUN bundle install --without development test
```

Then we declare what port we can to expose for traffic:
```
EXPOSE 3000
```

For Dockerfiles the ```CMD``` command is the command that runs your app and there can only be one of them (technically there can be more than one but the last one will always override the rest). There is a funny syntax here but the strings separated by the commas will result in the whole command being run like so: ```rails server -b 0.0.0.0```
```
CMD ["rails", "server", "-b", "0.0.0.0"]
```

There is a caveat here. This would not allow you to run a fully functioning Rails application. There is no database connection being made here. I've built this ```Dockerfile``` with the explicit end-goal of hooking it up to AWS so I'm not including a ```docker-compose.yml``` file where you could create a pre-made PostgreSQL image.

But as far as setting up a ```Dockerfile``` goes, that's it! Now in order to hook this up to an AWS Elastic Beanstalk instance we need to do some more things but let's first talk about what Elastic Beanstalk is and what it helps you do.

On the introductory page for Elastic Beanstalk Amazon sums up the service:
"AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker on familiar servers such as Apache, Nginx, Passenger, and IIS."

If you've ever used Heroku before, Elastic Beanstalk is a very similar service. When you use EB (Elastic Beanstalk) you get the added benefit of taking full advantage of AWS's powerful scalability features (like load-balancing and auto-scaling) as well as being able to easily hook into a variety of enterprise-grade services that AWS offers. You can build your application without having to worry about all the complicated infrastructure that supports it.

Now let's explore how we hook up our Docker application to AWS.

First, we're going to host our Docker image to Dockerhub. Go ahead and create an account on there (it's super easy). Once you've made your account you can create a local Docker image and push it to your Dockerhub account so that EB can pull your image when you're ready to have it hosted. Before you can do that though, you have to install the Docker CLI tools so you can login to your Dockerhub account and push to it. Once you've created your ```Dockerfile``` you need to build your image. ```cd``` into the directory with your ```Dockerfile``` and run the following command.

```
docker build . --tag latest
```

Docker will begin building your image with the tag "latest". Once you've built your image you can see it by running the command:
```
docker images
```

From here you can push your image to Dockerhub, follow the Dockerhub docs here for instructions on how to push your image to Dockerhub! [read this](https://docs.docker.com/docker-cloud/builds/push-images/))

Once you've pushed your image up, we need to create a a ```Dockerrun.aws.json``` file that tells EB where to look for our Dockerhub image. It's really simple, my ```Dockerrun.aws.json``` looks like this:

```
{
   "AWSEBDockerrunVersion": 1,
   "Image": {
       "Name": "akiraandy/forum",
       "Update": "true"
   },
   "Ports": [
     {
       "ContainerPort": "3000"
     }
   ]
 }
```

Here I'm declaring what Dockerrun version I'm using. Then for ```Image``` I'm pointing to my account on Dockerhub and telling it which image to use on my account. ```Update``` instructs Elastic Beanstalk to check the repository, pull any updates to the image, and overwrite any cached images. For ```Ports``` we expose the same port that we exposed in our ```Dockerfile``` which was port ```3000```. Now we can setup our Elastic Beanstalk instance!

If you haven't already, you should register for AWS. Everything that we're doing can be done on the free tier so no need to worry about incurring a lot of costs. We need to create a new application so give your application a name. For the "Base Configuration" platform, make sure to select Docker and for application code go ahead and start with the sample application code, we'll be able to override that application easy in the next step.

Once your EB instance has been created successfully, we need to replace the current demo application with our own! Click the "Upload and Deploy" button and choose your ```Dockerrun.aws.json``` file to upload (make sure you've filled out the JSON file with your own image!). Once you've submitted your image, wait for EB to upload your image. Once that's completed, you've officially got your application running on Elastic Beanstalk! From here you can create an RDS instance and hook up a database to your application. That'll be for another blog post though. Happy coding!
