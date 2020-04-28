---
title: Write a Telegraf plugin
seotitle: Write your own plugin to write data to Telegraf.
list_title: HTTP input plugin
description: >
  Add description here
menu:
  v2_0:
    name: Write a Telegraf plugin
    parent: Use Telegraf plugins
weight: 202
---

for a lot of you out there, I think, maybe, you’ve taken a look at what we have, and there’s something specific to your use case that you need, or you want to make modifications to existing plugins, or something like that.

## Intro

 - open source
 - can act as an agent or as a collector
 - over 140 plugins. most are community contributed
 - specifically written to handle metrics that have tags, as opposed to metrics that are uniquely names-based
- Written in Go. <will need to update with execd info>
- And of course, if you’re writing something, if you’re working on this, you can reach out for help either on our community site, or on GitHub issues, and one of our engineers, or some of the dev op team will get in touch with you and help you with that.


## Plugin architecture

- Modular: functionality in Telegraf, inputting data, outputting data, whether you’re collecting it from a system, or a database, or the network. Can be easily swapped in, one for the other.

- The way that we do that is using  Go interfaces. Each type of plugin has an interface associated with it.
  - Contributing.md: detailed guide about various types of plugins and their interfaces.

- Each type of input plugin has a well-defined interface that needs to be fulfilled: entire plugin architecture is built around these interfaces.
you go in, write a few files, fulfill those interfaces, you’ll have a working plugin that does what you need it to do.

### Input plugins

-  Used for receiving data or pulling data from variety of sources.

- Input plugin is defined as something that is executed continuously: ex., runs every 10 seconds

- As metrics come into it, it collects them and prepares them for output.

- Example of what input interface looks like.

- Three methods:
  - two for generating config files
  - `gather`: processes and receives the data.
    - Accumulator that runs every time the input plugin runs.
    - Takes any metrics that it's gathered and send them into rest of Telegraf application.
  - `SampleConfig`: returns the default configuration of the input.
    - Might include IP address if you’re reaching out to a network instance, file name if you’re processing log files, etc. <!-- verify that `SampleConfig` is one of the methods -->
  - `Description`: returns one-sentence description of the input.
    - So you can easily identify what each plugin is doing in the configuration file.  <!-- verify that `description` is one of the methods -->

---start tutorial---

## Intro
- Need latest version of Go
- Don't you need Telegraf? lol


 - Demo plugin called Trig.
 - Writes cosine and sine waves to the database.

## Get the source code of Telegraf.

You can use Go’s built-in tooling to do this.

Type into your command line:
``go get github.com/influxdata/telegraf``

This will pull down the source code into your Go graphs, which is your Go work space. All programming done in Go is usually done within the same work space. These tools automatically write your call down source code and build packages within that work space so that everything you’re working on knows about each other, and knows about dependencies, and things like that.

## Change directory to `GOPATH`
Change directory to “GOPATH,” which will be filled in by the [inaudible] `github.com/influxdata/telegraf`.

## Create a new branch from `GOPATH` directory and check it out

Once you’re in that directory, you can create a new branch and check it out using `git checkout -b mySweetPlugin`.

## Run a `make`
That’ll actually go through the build process and make sure that everything is configured correctly, and that you have all dependencies, and your environment is set up. And at the end of it, it’ll create a new Telegraf binary at GOPATH/bin/telegraf.

## Run the new Telegraf binary at `GOPATH/bin/telegraf`.

./telegraf (or whatever)

## Start making the files that you’re going to be working on.

1. CD to `plugins/inputs`.
2. Make new directory called `trig`.
3. Within that directory, create new file using `touch` command:
  ```
  touch trig/trig.go
  ```
4. `add boilerplate` to the file above.
5. `cat trig/trig.go` to make sure you've put it in the right place <!-- what does that mean -->

## Walk through go file of boilerplate

The first lines that you need are import statements. These are importing data from the rest of the Telegraf application, as well as some information about the various inputs.

  - Define a `struct`, which is the main data structure for this trig object. <!-- this is one of the import statements?  -->
  - And then we’ll define aa number of methods that get bound to that struct.
    - `SampleConfig`.  Sorry, I switched that. The SampleConfig is the actual data that’s going to go into your configuration files, and the next method
    - `description` is the simple one-line description of what the plugin actually does.
    - `gather` method. So we’ll put that in there as well.

## Function called `init`. which is called a Telegraf application itself at startup.
- Registers the plugin that you’re working on with Telegraf, so that it knows that it exists, and that it can call it, and that it can use it.
- Without this init function, Telegraf wouldn’t be able to find the plugin that you’ve created.
- In order for that init function to be called, Telegraf needs to know about the plugin that you’re creating. There’s a file called `all.go` within `input/all` in the `plugins` directory. And this just contains the list of all the plugins that have currently been contributed to Telegraf. Telegraf will scan through this list, it’ll find each of the individual plugins, and it’ll import them into the main Telegraf package. This makes sure that they can run after starup. Without this line, Telegraf will not know about your plugin, and it won’t be able to run it. So make sure that you add that.

Noah Crowley 00:26:40.696 The next thing that we’re going to do is add any configuration variables that we might need. This goes back to the idea that Telegraf can dynamically construct your configuration files by aggregating the configurations for all of its plugins. So here, in this example, you have “var TrigConfig”, and within that variable, you have “amplitude = 10.0”. This will set the amplitude of the wave form that we’re generating. If this were a plugin that were accessing resources over the network, we might add a default IP address there, maybe localhost. At the very least, we would define a space for the user to add that information themselves. But localhost is usually a good way to go for networking plugins. If it’s going to be reading files from the disk, again, that information can go there. If you want, you can actually put that variable within the SampleConfig function itself. But we like to just leave it outside for stylistic purposes; it makes it a little bit easier to read. So we have the variable here. We have “amplitude = 10.0” and we’re returning that variable when the SampleConfig function is called.

Noah Crowley 00:28:09.256 We’ll add a simple description of our plugin to go along with those configuration variables. The description is included above the plugin configuration in that configuration file. In this example, “inserts sine and cosine waves for demonstration purposes” and then you’ll set the amplitude comment, and then “amplitude = 10.0” within your configuration itself. So let’s test out the config stuff, even before we get to writing the actual code for the plugin itself. We can rebuild Telegraf by typing “make” from the root of the Telegraf directory. That’ll build the new binary and put it into your GOPATH/bin. And from there, we can type “telegraf” and pass it a few arguments to generate a configuration file, specifically, for our trig plugin. So those arguments are the first in SampleConfig, which will tell Telegraf to generate a configuration file. The second is the input filter, which is which plugins you want to add to that configuration file. And the third argument is output filter, which is which plugins on the output side you want to add to that configuration file. And then the debug argument just gives you a little bit more information as Telegraf runs. So you type that into your console. You should see an output that shows a bunch of data for the default Telegraf configuration, and then at the very end of it, you should see a section for inputs with the information that we’ve just added to our trig plugin. So it should have the one-line description, followed by the configuration variables.

Noah Crowley 00:30:10.841 Now that we have our configuration set up, we can start adding properties to our struct. Sorry, this is actually the wrong code example.

[silence]

Noah Crowley 00:30:49.520 Okay. I’m sorry about that. The slides are a little out of order. But let’s talk again about—all right. So we’re going to have to come back to that.

[silence]

Noah Crowley 00:31:32.164 Okay. So I can’t find the slide at the moment. But basically, we need to add a property to our struct that will actually hold the configuration variables itself. So right here, as we generated our configuration, part of the configuration is to have that amplitude in there. Our trig application is going to need to have a variable to store the state of the plugin between the collection intervals. So the amplitude is going to define how big the wait is, and the state is going to define where the point of the wave is at any given instance in time. I’ll come back to this. I think there’s a slide at the end with all the code; and I’ll point this out when we get to it.

Noah Crowley 00:32:22.827 So the next thing that we’re going to touch on is the Gather function. The Gather function is at the heart of the plugin. It’s run every time Telegraf collects metrics. So within Telegraf’s configuration, there’s something called the interval. The interval defines the amount of time that elapses between each one of the gather function. So by default, I think it’s up to 10 seconds. You can find it under the agent heading. And then under the agent heading, there will be a variable for interval 10 seconds, and you can change that to whatever you need it to be. There’s a second variable in there called flash interval, which is how often Telegraf writes that data out to the database, or the message dealer, or wherever you happen to be sending it. So one thing to keep in mind was that your collection interval should always be lower than your flash interval. So you want to be collecting data at 1-second intervals, but then potentially sending it up to the database at 5- or 10-second intervals, or something like that. Within the configuration, you can define different intervals for different plugins. But the most important part is that, every time that interval elapses, the telegraf.Accumulator.AddFields function needs to be called within your Gather function. So your Gather function is going to run, it’s going to do whatever work is required to gather that data, and then the telegraf.Accumulator function is to be called within that to add a new point for InfluxDB, or for whatever time series database you end up using.

Noah Crowley 00:34:12.845 So this is our Gather function for the Trig example. It has two main properties. We have the sinner function, which computes a sine wave based on, say, this X variable here, which is what I mentioned just a minute ago; which is what’s storing the state of the wave between individual points, and then multiplying that by the amplitude that we provided earlier to determine the height of the wave. We need to call each individual field to make sure that the measurements are tagged appropriately—sorry, the measurements are stored in the appropriate field. And then we need to define which tags we’re going to add to those fields. So once that’s done, we can increment X, which is that variable which is storing the state between intermediate calls. And we can call the accumulator and add that information, these various points, into the accumulator. So AddFields, we have the fields for the sine function, we have the field for the cosine function, and then we have any tags that we ended up defining. Okay, that, we can “return nil” because this function is returning an error, and everything seems to be working all right. If this were a plugin that were gathering data from the network, or from a file, or something like that, you might have to a little bit more error handling here, rather than just returning nil.

Noah Crowley 00:36:07.361 Okay. I apologize, this is the starting slide. Got a little out of order. But within the innit function, we’ve defined that X variable within our struct, and in the innit function we’re setting the initial state for the X variable to 0. So the sine wave will start with an initial value, 0, and every time the Gather function runs, we’ll increment that X variable by 1.0. And that will give us our sine wave. So now that all that’s written, we want to compile and test our plugin. The first thing that we’re going to do is rebuild. We can do that using the “make”. After that, we’ll generate a new sample config using the same command as we generated before. This time we’re going to store it in Telegraf.conf.test. Anything with .test at the end of it is ignored by default by the gitignore in the repo. So that’s a good place to store your configuration files as you’re testing them out, if you don’t want them to get committed. And then we’ll go ahead and launch Telegraf using the configuration file that we just generated, as well as the debug option. You should start seeing data being returned from that command. Telegraf will start up. The appropriate outputs and inputs will be loaded, and you’ll see the agent begin to collect data every 10 seconds. So the configuration will be flash interval of 10 seconds, and it should start running immediately. And after 10 seconds have passed, you should see the first entry here gather metrics from one input, and the amount of time that it took to gather this metrics.

Noah Crowley 00:38:13.091 The next thing to do, to make sure that the data has actually arrived, if you’re shipping this to Chronograf and InfluxDB, the you can open up Chronograf and go into the Data Explorer itself. So this is actually a live version of Chronograf, that I have running on my own machine. Within the Data Explorer, you can come over here and select the Telegraf database and the Trig plugin, and you’ll see those two fields that we just defined earlier while we were writing our Trig plugin. We can select data from both of those, and you can start seeing that a sine wave is coming in to InfluxDB. And we’re appropriately saving and visualizing that data. If you’re looking to actually submit those for inclusion in Telegraf as the open source project, you’re going to need to write some tests. Tests are really important. They guard against regressions or changes in the code that might start causing it to output different kinds of data and break other people’s integrations. So getting tests in there is really important. For out sine and cosine generation, they’re a little bit trivial. We’re simply going to generate some data and add it to the accumulator. But again, if you’re writing a more substantial plugin, testing is a really important part of it. And we do require that tests be added before you can contribute the function to our story.

Noah Crowley 00:40:00.047 A couple of other requirements, if you’re looking to submit a plugin: you’ll need a README.md file. This should provide some information about your plugin: how it works, what it’s doing, enough so that if someone’s visiting the repo and is interested in using your plugin, they can get a good idea of how it works. We’ll require your LICENSE file be in there. We’ll require you to sign our CLA, which is our contributor agreement, just so that the codes can be guaranteed to be open source, and other people can use it. And then it’s really important for us to get a sample of the output and input format for the plugin. The reason for this is that, a lot of the time, the engineers that are working on Telegraf might not be particularly familiar with the particular application, or the way that you’re collecting data. And in order to verify that the plugin is working as expected, we need some examples to compare that to. So really important that you help us out a little bit on that. We really want to grow these plugins and the community to a point which is way beyond what any of our engineers could handle on their own, and getting help from the community in terms of how the plugins are supposed to work, and how these third-party applications export data, is a really integral part of that. But if you’ve done those three things: if you’ve signed our CLA and you’ve written a plugin, then you are, at this point, ready to create a pull request. And we’ll take a look at it. You might have a few suggestions on how the plugin could be improved or made better. But you are on your way to having that be a part of our open source application.

Chris Churilo 00:42:09.597 Yeah. So I made a mistake on slide 25, so should we go back there and just show them how to add the properties?

Noah Crowley 00:42:17.679 Sure. All right. So this is adding properties to the struct. Basically, what we need to do—let me see if I can just actually get the code open for this plugin. So again, this is the Telegraf repo. Definitely recommend checking it out. We’ve got a great contribution guide. But if you’re interested in the specifics, we’ll go into plugins, inputs. And we’ll take a look at out Trig plugin, which is actually already part of the repo.

[silence]

Noah Crowley 00:43:17.501 So what we’ve done here is, we’ve added a few properties to our struct. This is important because this is the data that is being used by our plugin. If it wasn’t added as properties to our struct, then we would have nowhere to store it; we have no information about it. So we’re adding two properties here. The first property is just X. That’s what I mentioned earlier, which is going to be used to record the state of the wave an any given point in time. And the second property called amplitude. So amplitude is going to be set using the data from the configuration variable. And the X is going to be set, initially, within our init function. And we’ll set it to 0 within the init function. And every time the Gather function runs, what we do is we increment that variable by 1. So again, if this were reading from disk, or something like that; if it were accessing resource over the network, you’d want to add any pertinent information variable to properties within the struct, and the interact with those properties from the Gather method, from the init method, or from wherever it’s appropriate.

Chris Churilo 00:44:47.295 Cool. You do have one question in the Q&A. You want to go ahead and read it out loud?

Noah Crowley 00:44:51.941 Sure. Would you mind reading it, because I’m not sure where it is?

Chris Churilo 00:45:05.045 Yeah. No problem. It’s no problem. So Davie Telesco asks: What would be the suggestion for getting data out of Node.js service?

Noah Crowley 00:45:15.740 So if that’s a service that you’ve written yourself, the best way to do it is actually to instrument the application as a developer. I’m not intimately familiar with Node.js. But with other web frameworks, and things like that—if you look at Flask, for example, Flask is a Python framework. It’s built on top of WSGI. And the best way to get metrics out of that is actually to instrument your application directly. So with the Flask framework, you can go in, you can add to middleware. And the middleware can start recording whatever information you feel is most valuable about the application. So if it’s a web application that you’re writing, you can start recording things like response times for endpoints, error rates, information like that. You can instrument it directly within the application, and then you can either send it to Telegraf via UDP, or you can expose it within your application as a Prometheus/metrics endpoint, and the Telegraf can read from, or something like that. So I believe Node is very similar in that way, and that you have middleware, and you can add your own code to the middleware. There might be some packages out there, particularly for Express, or something like that, for actually instrumenting the application. But once you have instrumented your application, and you have some code there to start generating the metrics that you want, you can do a whole bunch of things to get them in Telegraf: you can write them to a file, you can send them out via UDP, or you can expose them as a metrics endpoint that Telegraf can then go an check.

Noah Crowley 00:46:58.992 I think in terms of speed and performance, UDP is probably going to be the most performing way to get your data from that Node.js application to Telegraf. But it really depends on your environment, and how you’re working, as to whether or not that’s the best approach. But I’d say, generally, you want to instrument your application first, and then depending on your setup, you want to get those metrics from your instrumentation into Telegraf either via UDP or by writing to a structured log file, or by creating a metrics endpoint than exposing that.

Chris Churilo 00:47:37.672 So looks like you answered that problem perfect. So if there are any other questions, feel free to throw them in the Q&A chatter. Otherwise, if you have questions after, which often is the case, feel free to go to our community site, and Noah will be there with the rest of the team and be more than happy to answer any of your questions. Now you’re fairly new to the project. When you first approached Telegraf, based on that experience, do you have any tips that you might want to share with the people on this webinar? Things that, maybe, you did that you with you didn’t? Or maybe there would have been more efficient approaches?

Noah Crowley 00:48:21.789 I think the easiest thing to do with Telegraf is definitely to make sure you understand how your applications work, and to take a look and see whether there are plugins out there that are already configured for them. I definitely tried to go the hard route and start reading from some log files, initially, when there was actually already a plugin prepared for my application that I could just easily drop in and start getting metrics out immediately. I think in general, collection agents like this are pretty easy to get started with and get started using. So my recommendation is, really, don’t wait; put this into your system as quickly as possible. Drop in some plugins, start getting some data out of it, and the begin taking actions based on that data. I think it’s really hard to understand the systems that you’re using without actionable information about those systems. And once you start gathering data, you’ll get a better feel for how your systems work; you’ll get a better understanding of what you might need additionally, that you’re not currently collecting; and you can start making data-driven decisions about how you want to engineer your system in general.

Chris Churilo 00:49:46.222 I think that’s really good advice. You’re right, the more you can dig into it, then the better you will understand things. I think the other bit of advice that I would offer to our attendees today is that, don’t be shy about putting in your own contributions. And one of the things that we often find is, people will fault the project and make some changes, fix some bugs, then kinda just leave it at that. And it often turns out that there’s actually someone else that had already done that. So you know, it’s always good to bring back those changes to the main branch and share the work amongst everyone else. I can’t tell you how many times that I’ve introduced people to each other that have done the same changes. You get a bit of a chuckle, but then you realize, “It could have been a lot more efficient if we just all put them into the main product.” And as Noah has mentioned, it’s a lot of fun. It’s a great way to get started. So we encourage you guys to get started. If you do get started and you do run into some issues, Noah is actually going to be holding workshops at InfluxDays in New York. So if you happen to be in town, I would recommend that you pop over and get to know him face-to-face. And then really try to stamp him with some really hard questions [laughter]. I’d really love to do that for a change. All right. We’ll wait just for a few more minutes. If there are any questions, put them in the chatter Q&A. Just as a reminder, this session was recorded. So we’ll post it later on today. And if you happen to have a really cool project that, maybe, you want to even share with us, let us know. Noah is always looking for people to join his meetup in New York City, so we’d love to ask you to come in and present your solution as well.



### Output plugins

- Output example interface

## Processor plugins

- example interface?

## Aggregator plugins  

- example interface?
