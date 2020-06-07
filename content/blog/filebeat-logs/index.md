---
title: Filebeat (part of beats family)
date: "2020-06-16T00:00:00.000Z"
---
This is my first foray into reading code through open source repositories and write down thoughts. Hope to do this on a regular basis.

### How I got interested?

We use it internally to ship logs from machines to Kafka. As part of a recent Kafka upgrade, I ran into Kafka running into replication issues. Upon further investigation we found an older version of Filebeat might be the culprit.

### Why is it interesting?

If not for any reason, purely for the amount of customization it supports. I was also interested for long on how people code efficiency into daemon type of processes without overwhelming other stuff on machine and how do they build packages for different OS's. A good third reason is it is written in golang and I want to get my feet more wet there.

### Story

Filebeat is a package in family of beats family open sourced and backed by elasticsearch. [Beats](https://www.elastic.co/beats/) are termed as single purpose data shippers. Other interesting beat from the family is Heartbeat, which ships uptime monitoring data.

If you are interested in getting highlevel overview of what filebeat is and how it works, you should read [this 101 doc from elastic](https://www.elastic.co/guide/en/beats/filebeat/current/how-filebeat-works.html).

To do a simple tldr of that article, there are 5 modules and their responsibilities are as follows (takeb from one of the source files):

1. input: finds files in paths/globs to harvest, starts harvesters
2. harvester: reads a file, sends events to the spooler
3. spooler: buffers events until ready to flush to the publisher
4. publisher: writes to the network, notifies registrar
5. registrar: records positions of files read

Finally, input uses the registrar information, on restart, to determine where in each file to restart a harvester.

**Build:** Uses [mage](https://magefile.org/) to build modules. My one sentence understanding of mage is a transpiler from golang to make. Build file using golang is located [here](https://github.com/elastic/beats/blob/master/filebeat/magefile.go). At the end of day building for cross platform is nothing of a magic sauce apparently. You just need to know where to install various things as this [piece](https://github.com/elastic/beats/blob/master/filebeat/scripts/mage/package.go#L55-L65) of code shows.

```go
for _, args := range devtools.Packages {
		for _, pkgType := range args.Types {
			switch pkgType {
			case devtools.TarGz, devtools.Zip, devtools.Docker:
				args.Spec.Files[moduleTarget] = module
				args.Spec.Files[modulesDTarget] = modulesD
			case devtools.Deb, devtools.RPM:
				args.Spec.Files["/usr/share/{{.BeatName}}/"+moduleTarget] = module
				args.Spec.Files["/etc/{{.BeatName}}/"+modulesDTarget] = modulesD
			case devtools.DMG:
				args.Spec.Files["/Library/Application Support/{{.BeatVendor}}/{{.BeatName}}/"+moduleTarget] = module
				args.Spec.Files["/etc/{{.BeatName}}/"+modulesDTarget] = modulesD
			default:
				panic(errors.Errorf("unhandled package type: %v", pkgType))
			}
			break
		}
	}
```

There is mot much more to talk about build(I am in no way talking down about the task, but library ecosystem makes it little easier). Rest of the stuff is about how you generate modules specific to target platform and docs + config. Another interesting thing I started observing is, in golang you line up your tests either in same file as in case of build file [here](https://github.com/elastic/beats/blob/master/filebeat/magefile.go#L178-L180) or alongside source files as you can observe through out repository.

## Deets on app

I am new to golang, but I think common pattern in golang for atleast daemon processes is to have a `cmd` module as entry point. [Cobra](https://github.com/spf13/cobra) is used for command line parsing. I have seen that once before for an internal library and seeing same pattern in this repository. And cmd module will instantiate beater module, which leads us to this [place](https://github.com/elastic/beats/blob/master/filebeat/beater/filebeat.go#L74).

### Startup

Beater module is where app gets initialized. It reads through config, finds what the inputs and outputs are. While reading the source code, found out that you can also give inputs config via stdin which is a good pattern to have for all library applications. You are well off supporting this use case for runtime debugging or during development instead of changing config, restarting application and checking to see if it works. Once config is read, it will  Also found registrar as a cool pattern to maintain state. Basically you create whatever counters you want anywhere in the application, pump them to registrar and update them on a regular basis. This is akin to an inmemory database. You can find registrar source code [here](https://github.com/elastic/beats/blob/master/filebeat/registrar/registrar.go). I will try and delve little more deeper on registrar later in the post. **TLDR**: startup includes reading config, setting up pipelines to move data from local to elastic search and instantiating in memory database in form of registrar.

### Input

Module that runs over input part of config, scans for files on a regular basis or once depending on scan frequency and updates registrar with file list. Two things of interest from language capabilities side for me here is the use of golang channels to signal exit (or [done](https://github.com/elastic/beats/blob/master/filebeat/input/input.go#L68)) and sync [wait group](https://golang.org/pkg/sync/#WaitGroup) as a mutex for file scan. Wait group waits for a group of co-routines to complete and act as a synchronizer. And as for code structure the way different kinds of inputs are handled is interesting. They follow the [factory pattern](https://github.com/elastic/beats/blob/master/filebeat/input/input.go#L78-L79) to generate inputs based on type.

### Harvester

A harvester as name indicates is basically the module that collects data that needs to be shipped (harvesting data). It is responsible for keeping track of file state changes and listening for new updates. Harvester interface basically has id, run and stop. You can right your custom harvester, for example you are trying to ship data from say postgres to kafka or s3, you can extend this interface and write your own harvester. This package also contains forwarder and registry. Forwarder is an interface that defines how events are forwarded to our sink and registry manages state of all forwarders. There is nothing much interesting in terms of code patterns here for me.

### Spooler

I had this header after reading the startup code. But looks like this part of filebeat architecture is deprecated and responsibility is distributed between harvester and publisher (majority responsibility went here) according to this [change](https://github.com/elastic/beats/pull/4644).

### Publisher

Publisher package lies outside of filebeat submodule. Its a common package across all beats libraries. Takes events from harvester and pipes it to output. Its not much interesting for outputs like kafka, where it is pretty simple which is event driven. This gets little more interesting for elasticsearch where you can customize it a lot. You can define which index you can pipe events to or even better define a pipeline. A pipeline is basically a set of stages where you can do data processing in different stages before putting data in elastic search index. Even better you can select your pipelines based on the input event data, like as example states move one index for warning events and another pipeline for info events. Publisher/pipeline is the meatiest part of whole filebeat and it deserves its own dedicated time for analysis and post. I will try to keep it to minimum and focused.

Pipeline consists of clients, processors, a queue, an output controller and output. According to documentation queue is the central thing to whole of pipeline. Queue provides typical queue capabilities pushing, batching and batching events. And output controller load balances output clients. My favorite part is processors, which runs async in pipeline as go-routines. Go-routines are super lightweight. Processors help morph the data in the way we need without having to write another stream processor or deploying lambdas.

As far as code is concerned I liked the ack parts of pipeline. I am a big fan of kafka, would love to get to its source code sooner than later once I get good at this. I like the ack part of this code. You can look for yourself the code [here](https://github.com/elastic/beats/blob/master/libbeat/publisher/pipeline/pipeline_ack.go#L130-L149). The part I want to highlight is following. 

```go
for acked < count {
		var msg eventsDataMsg
		select {
		case msg = <-p.events:
		case msg = <-p.droppedEvents:
		case <-p.done:
			exit = true
			return
		}

		if msg.sig != nil {
			signalers = append(signalers, msg.sig)
		}
		total += msg.total
		acked += msg.acked

		if count-acked < 0 {
			panic("ack count mismatch")
		}

		switch p.mode {
		case eventsACKMode:
			data = append(data, msg.data...)

		case lastEventsACKMode:
			if L := len(msg.data); L > 0 {
				data = append(data, msg.data[L-1])
			}
		}
	}
```

It looks quite simple while handling all the usecases which pipelines supports in terms of acks (there are 4 modes). There is a companion class that goes along with this - [acker.go](https://github.com/elastic/beats/blob/master/libbeat/publisher/pipeline/acker.go). Now that we have talked about acks, we can build ourselves towards the master piece for all of pipeline which is queue implementation. Like we talked before, queue is the central part to whole pipeline. There are two queue implementations. One is in memory queue called memqueue and other is on disk queue called spool. I will concentrate mainly on memqueue. If you want to explore this for yourself this is the starting [point](https://github.com/elastic/beats/blob/master/libbeat/publisher/queue/memqueue/broker.go#L74). When queue is created, a new broker which co-ordinates all the events on queue is instantiated. A broker (like kafka one), takes care of handling requests to queue, storing data and working through acks. I get a feeling that if you want to implement a queue of any kind, this code can serve as template for implementation. I would recommend to look this part for yourself. I want to comment on almost every single line in this [file](https://github.com/elastic/beats/blob/master/libbeat/publisher/queue/memqueue/broker.go). While you are there please enjoy a serving of [ringbuffer](https://github.com/elastic/beats/blob/master/libbeat/publisher/queue/memqueue/ringbuf.go#L26-L38). I will probably comeback to explore this module more in depth once I read a little bit more about go channels. 

### Registrar

After spending time in the queue section, this part feels lot more dull in comparison. Registrar as we talked before manages state of the application. But I was thinking it is responsible for even keeping track of events which seems to be responsibility of publisher. Its a farily straightforward module which handles file discovery and keeps track of what is read from the files along with co-ordinating graceful exit. 

### Todo Items

- Comeback to explore publisher
- Learn golang channels in depth to help understand code more deeply
- Find out performance characteristics
- Figure out if going via kafka to s3 is more efficient or going via logstash to s3
- Go and use some of the config discovered along the way at work

### Other things I learned along the way

[Graphviz](https://graphviz.org/) - Graph visualization software. I have been scouring for something like this for salt state. Without knowing name it is shooting in dark and after knowing it, I found what I wanted instantaneously.

[Flatbuffers](https://github.com/google/flatbuffers) - Thrown into interesting and famous packages I didn't knew bucket.

While exploring mage found [Hugo](https://gohugo.io/), which has got 45k stars on github. Will give it a try for next website building adventure.