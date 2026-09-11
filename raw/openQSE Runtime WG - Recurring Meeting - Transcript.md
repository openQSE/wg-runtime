# openQSE Runtime WG - Recurring Meeting - Readable Transcript

Source: `openQSE Runtime WG - Recurring Meeting.vtt`
Transcript span: approximately 00:10 to 59:25

Note: This is a readability conversion of the automatic caption transcript. Speaker labels and technical terms are preserved as captured where possible, but some names or terms may reflect captioning errors.

## Speakers Detected

- Shehata, Amir (88 turns)
- Cambridge SuiteB GREAT OUSE (6) (34 turns)
- Bertels, Luke (17 turns)
- Tang, Xulong (16 turns)
- Eckert, Yasuko (12 turns)
- Srikar Chundury (8 turns)
- Abdur Rahman Hatim (3 turns)

## Transcript

### 00:00-05:00

**[00:10-00:12] Shehata, Amir:**

So what do you guys think? What's the...

**[00:18-01:17] Cambridge SuiteB GREAT OUSE (6):**

I think our stance is a bit easier to have because we already have a tool that we are working on. So we are definitely, at least in our group, I think it doesn't make sense to talk about a really low level device runtime. I think there's also quite some overlap with... one of the existing groups there. So I think we should be at least one level above this. And I think then it's debatable which is which. And at least in my opinion, I wouldn't make the separation between Um... the application runtime and then having this co-location runtime or however you might refer to it.

Because I think one runtime can take care of both of this, at least it should be possible, but I'm not sure.

**[01:19-02:21] Shehata, Amir:**

When you say, in my mind, application is the one thing that, like application runtime and co-location is the same thing. In my mind, it's not two different things. So, the idea is, in okay, so maybe compare what do with what I was thinking. So, in it's a workflow manager, as far as I understand, it runs on the head node, for example, in a cluster.

It takes a description of what the job is, and then in the description you have, like, you know, classical and... quantum sites, and then it takes that description and then it goes and reserves using Slurm or whatever, it reserves the classical resources, runs that job, gets the results back, then you know, like moves on to the next step and so on, and then you can have like this DAG diagram that you like kind of compose the different workflow. So that's what I understand term crisis that. Overall, correct.

**[02:22-02:22] Cambridge SuiteB GREAT OUSE (6):**

Yeah, yeah.

**[02:24-03:00] Shehata, Amir:**

That, in my mind, the application runtime is assume that you have one of those steps in is actually a combined quantum classical operation, right? And then you offload that, but instead of just reserving quantum or classical, now you're reserving both. You're reserving like 5 classical nodes and say 2 quantum nodes, assuming that you have more than once, more than one. And then the question becomes, how does the application manage those resources that it has access to?

**[03:03-04:23] Shehata, Amir:**

So if we look at it right now, the application has MPI, for example. It can use the MPI library to manage or run some MPI process. But when it comes to quantum, It's just, it will have to use the back end. So it will have to create 2 back ends, and then it will have to understand this back end is of type something, this back end is of type something. I'm going to generate a task, and then I'm going to assign that task to back end one, assign the second task to back end two, et cetera. From an application perspective, maybe they don't care about doing that, like, right?

Like, there is no need for them for an application to developer to worry about the different resources. Maybe he cares only about a pool of resources, and then I just want to submit a job. I don't care where it's going to run. just run it and give me back the results. So the application runtime I was thinking about is more at that layer. It manages the resources, that it provides an abstraction for that hybrid application to decide to look at it as just one. pool of resources that I submit jobs to and it runs somewhere, then I get results back.

**[04:29-05:46] Tang, Xulong:**

So, I mean, so if I recall from last week discussion or previous conversation, so the two runtimes, the big separation between them, in my mind, I mean, because the compiler, right? So whether you're taking the compiler before the compiler or after the compiler. Work, right? So, what Muhammad discussed about space RT, most of the parts at least to the development tested for planning internally and open source or hand to Oak Ridge and rise shortly is the after compilation. That's what Muhammad mentioned. So, after compilation compiler does all the graph. analysis, data flow, control flow, and then gave to the hardware.

And the hardware will be able to start dispatch, schedule, and all this. This is basically the runtime mode towards the device or the second level you're mentioning, right? So the other runtime is before the compiler, when we submit the application, then you have a runtime there to manage the... resources and also you have a two pool of tools in which you can call and some of them are compiler modules you can use that for circuit or whatever optimizations you want to apply. That's the other runtime. Am I on the right impression on this?

### 05:00-10:00

**[05:47-06:21] Shehata, Amir:**

I think so. I think the part that is a little bit hard for me to wrap my head around is the after the compiler bit. So after the compiler, what exactly do you guys see space RT doing? That's the part of not fully grokking, because after the compiler, the compiler will compiler for a specific hardware. Right, it will compile for a continuum, or it will compile for, you know, whatever. like back end it's compiling for. And then what is the role space RT at that point?

**[06:23-06:35] Tang, Xulong:**

I mean, for even for compiler, you have a multi-level of MRR, right? So, intermediate representations, right? There, right? So, for space RT,

**[06:32-06:32] Shehata, Amir:**

Mm.

**[06:35-08:23] Tang, Xulong:**

if I understand correctly, I'm also working on that, testing it to it provide automative. or you can sync at high level equivalent API tools compared to HIP. And HIP is a one-to-one mapping from CUDA. That's basically CUDA runtime. That's basically kind of the relationship. And the better thing of SpaceRT compared to HIP is that it's more native using the hardware. So at high level, I mean, what do you give space RT could be a kernel graph, I mean, captures the dependencies, like Muhammad mentioned last time. And all this dependency information comes from the compiler.

So space RT does not do like kernel fusion, this kind of compiler approach, kernel fusion. or kind of kernel internally the mapping of the threads or this kind of thing should be done by auto tuner or compiler work. But Space RC does not do that. But once you have the graph, how you launch this kernel, how to put them into task queue, how to fetch that. That's some of the basically it's more like a at device level how you schedule the task on the hardware platforms. So that's where it tries to optimize. For example, even you compile for one GPU, one per specific type of GPU, right?

So how are you going to launch that kernel? How you prepare the kernel parameters? Should you create from scratch or you have a parameter warm up face which can pre-prepare all the parameters and so forth. So those points are what space RT runtime is doing in terms of GPU. That's my. If that aligns with what your understanding of that.

**[08:26-09:01] Shehata, Amir:**

I understand from the GPU perspective, I'm just trying to like map that onto the QPU. That's the part that I'm having. Like I understand that this, it seems to me that this is targeting QEC specifically, right? You have, you want to be able to read syndrome data off the QPU and then feed it into some GPU to do some decoding and then take the results back and send it. send it like corrections to the QPU. And you want something in the middle there that manages this back and forth data and you want it to be efficient. That seems like the role of space RT to me.

**[09:02-10:39] Tang, Xulong:**

It could took that bro, but it's not designed just for QEC. I mean, QEC can use that. That's why I would recommend at the beginning stage, we don't put the QEC part as one of the runtime layer or the software stack for the quantum. So the QEC should be a separate thing. other than the runtime and compilation stack we're developing in this group here. That's my understanding, because QUC has specific requirements on the latency, the backlog, free for the single data, everything. And if you launch on GPU, you can definitely use the space RT or achieve the using the low latency mode for that.

But it's not a runtime design just for QC requirements throughput or latency. In other words, you can think QEC is like a one use case for the runtime. I mean, you give constraints, what do you want to meet? You want to meet latency, you want to meet throughput, you want both. And then the space RT will have different mode, which have the runtime knobs, we call it knobs inside, to try to stride a balance between your targets. It cannot achieve all of them if you give a very strict requirement all of them. But it can try to try to optimize our strata trade-off based on your priority metrics.

That's something actually from the quantum application, we also provide constraints, right? That constraint will all the way down to the appropriate layer and then put constraints for that layer. It's a compiler or... Runtime or execution or resource management.

### 10:00-15:00

**[10:41-10:50] Shehata, Amir:**

So to avoid, sorry, I like, I'm trying to learn the lesson from last time and to avoid going into the weeds right now, right?

**[10:48-10:48] Tang, Xulong:**

Mm.

**[10:50-12:12] Shehata, Amir:**

So there's a lot of details here that I think needs to be clarified and maybe can be clarified by like if you guys show us an actual example of running code. that interacts with a QPU through Space RT, I think that will clarify a little bit more what Space RT's role is. But if we take a step back and we say, okay, we have this runtime working group and we identify that there is a device runtime, but there's also this higher level. you know, resource orchestration runtime layer. And then you have a third layer on top of it, which is the workflow manager.

In this working group, we can tackle all of them, but like if we would prioritize, which one would prioritize first? And it feels to me, at least in the current state of work that we like work. top down. So we look at the workflow, we look at the runtime layer, and then we look at the device after. And we don't have to do it like, you know, sequentially. We can obviously like you like AMD is already working on that so that they can continue working along that path. But. I just don't want to lock us into that particular path and not explore the rest.

That's I guess that's where I'm standing right now.

**[12:12-13:20] Tang, Xulong:**

I fully agree with that strategy. And I do think from top down, because essentially, I mean, if you think at a very high level, you can describe your application in some type of graph, right? And that's basically the object model you were mentioning at the beginning. And those graphs will be further zoomed in for each node, the one, for example, at the beginning, the node may be very coarse-grained, and you zoom in the node when you go to the next layer or the middle layer, and then when you go to the device layer, you zoom in again, it becomes more complicated.

And I fully agree with this is a top-down approach, starting from the application. and what requirement, what application developer really cares. I mean, we don't want to put a lot of constraints or complications to the application developer. What they care, and that will define what the top layer does. And then we talk about the second layer interaction with the first one, then go to the. Uh, bottom runtime layer, so that, that, uh, that, that's definitely the proper approach, I would say.

**[13:23-13:24] Shehata, Amir:**

OK, so I think.

**[13:24-13:26] Tang, Xulong:**

So, my, yeah, go ahead.

**[13:25-13:28] Shehata, Amir:**

But, no, no, finish your call.

**[13:27-14:28] Tang, Xulong:**

Yeah, so my understanding from, I mean, from the group discussion also the workshop is that application developer now most of the time for quantum algorithms, they will just use Qiskit, for example, the quantum SDK, right, to write their circuit model and just use the circuit model, for example. to write their algorithm. And just for the quantum part, the QPU part, if I understand correctly. And if that involves classic and quantum, it'll be written in two, think about even just two files, right?

So then clear, deter, decide what should be the. data or information exchange between these two files and then compile into two binaries, one running on GPU, one running on the QPU, and use some pre-processing, post-processing to merge without back. That's just my very naive understanding if that's the case. And then basically... We can talk about what Qskit.

**[14:33-15:00] Tang, Xulong:**

programming apps and programming model they're using for their quantum part. And then we basically can from there start to look at what should be the first layer of the resource when we combine these two together instead of putting separate files and defining information. Is this something that the kind of intuitive start that's in my mind just? Yeah.

### 15:00-20:00

**[15:02-15:05] Shehata, Amir:**

So I'll open them to the rest. I don't want to keep talking. What do you guys think?

**[15:08-16:45] Cambridge SuiteB GREAT OUSE (6):**

Yeah, I think the instinct about you have the highest level graph and you zoom in and then there's this low level graph and each of those kind of correspond in some ways to... I mean, obviously not directly, but can go all the way down to the device layer or the application layer as we were talking about. I think that approach makes a lot of sense. It is definitely true that a lot of people develop the stuff in the circuit model in Qiskit. We are doing things a little bit differently at Continuum just because we've got like... the real-time control with Gothi stuff.

And that does also kind of correspond a little bit to this idea of graphs and graphs and graphs. I mean, like hugger, the hierarchical graph representation that we've developed does have that like multi-layer stuff. And I think LLR also has similarities in that area as well. So there's some overlap to what. we're talking about. Yeah, I I think that makes sense though, but it's not, there's certainly a lot of efforts out there beyond just Qiskit.

I mean, the the thing paying lane put out as well the other day, this like backline thing is probably also relevant to the working group, and with Andy as well, because that's like the application layer, from what I understand, is in like the... The second layer, uh, whether we going up or down, um, and it's, I think that's MLIR based, but yeah, having like... There's certainly a possibility in the future as well that there could just be a single shared representation that does describe the workflow, the application, and like the error correction stuff all the way down as well, but that's quite speculative.

Anyway, sorry, I'm just kind of talking now, so...

**[16:45-17:04] Shehata, Amir:**

No, no, does it make sense though? Like, if you guys are doing work like in that area, obviously, or would it make sense to like present that to the group so that we can at least get that good understanding? Like, we heard the third price like presentation, but we don't know like the lower layers and you know the zooming in that we're

**[17:02-17:02] Cambridge SuiteB GREAT OUSE (6):**

Mmh.

**[17:04-17:08] Shehata, Amir:**

just talking about. It will be nice to know how you guys are

**[17:07-17:08] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[17:08-17:11] Shehata, Amir:**

doing it. In the in continue.

**[17:11-18:50] Cambridge SuiteB GREAT OUSE (6):**

Yeah, we probably could, I think. I think some of the things I'm talking about are speculative and possibly things that are things I would think about necessarily, rather than things that the company has endorsed as a direction. But certainly, like talking about the hugger would be pretty nice and guppy and how that works on the sort of runtime-ish layer. And then how you potentially could extend that up to working at the workflow level as well is maybe interesting. But I'm just certainly interested to explore a lot of things that other people are doing in that area. And...

And maybe a completely different point, because it was mentioned again that we have this compilation layer in between. This is one of the, I think, most trickiest problems we have to solve somehow, because it's very easy to draw the boundary at the compilation. The problem is currently, I think most of the programs are in a state where it is indeed true that quantum and classical code can be separated, and we don't have quantum code that is generated at runtime, so to say. So we have this clear boundary.

Once we have a program that needs to invoke a quantum compiler at runtime, this boundary goes a bit away and it becomes very muddled what is now derived runtime, what is now compile time, what is now application runtime. So we have to be very careful when drawing this boundary here, I think, because it's very easy to say, okay, we have this separation, but I think in Reality tends to fall apart quite quickly, I think.

**[18:51-19:55] Shehata, Amir:**

So that actually brings one thing I was just thinking about right now. Should we look at this from the short term versus the long term perspective? Like in the short term, what can we do? What can we deploy on a test bed, like Oakridge's test bed, for example, with the equipment and the stuff that we have versus, you know, future looking? what, how, how do we need to evolve the environment to work in the future?

Because I feel like in order to show some like concrete progress, it will be nice to actually, you know, get Terracrist working with some runtime layer and then actually get something running on a, on a, like, you know, on a test bed. To prove the concept that you know there is this better method of like exposing to the application developer rather than just using the back-end mechanism that already existed in the in. all the different SDKs, you know, like ticket, Qiskit, etc.

**[19:58-21:35] Cambridge SuiteB GREAT OUSE (6):**

I mean, I think we'd be in favor of that. Yes, we we are doing similar things already as well, and yeah, certainly finding pain points that way is very helpful, just for development, and the problem with the with the long-term vision is that error correction. becomes a huge unknown because so if in the long term a lot of these real-time operations become a lot more relevant.

This is better than something like the stuff AMD is doing becomes more relevant because it makes sense to have this control over these closer interactions. while in the current state-of-the-art or in the current hardware generations, there is not much you can do with real-time control yet because the current times are still very short. You cannot, the only thing you can basically do is error correction, but then the devices are not there to do like a full-scale. error corrected thing. So I think this is very hard to get right. And I think we should try to focus on the short term to get the architecture going.

And then once the picture for error correction becomes more clearer, try to integrate this. And I think this is then maybe also the point where these separations become less clear and we get this like one true runtime that is then consists of multiple runtime levels. I guess there's also, sorry to jump in very quickly,

### 20:00-25:00

**[21:33-21:33] Tang, Xulong:**

Yeah.

**[21:35-21:45] Cambridge SuiteB GREAT OUSE (6):**

is it do we want to produce a runtime or runtime spec? I guess. So the idea is to produce a spec and then we can interchange components, maybe makes sense as well.

**[21:46-22:18] Shehata, Amir:**

Yeah, I think that's a good question because what are we, we need to kind of hone down on what we want to produce as part of this. And in my mind, at least, I feel it's difficult to actually figure out what we want to produce from a specification perspective until we have like a top to bottom software stack that is actually running something. And then we can sort of like start examining it a little bit more closely and finding the common, you know, surfaces that needs to be more, you know, standardized.

**[22:20-22:20] Tang, Xulong:**

Yeah.

**[22:21-23:22] Shehata, Amir:**

So my, here's, I'm going to throw up an idea since Luke is on the call. So Luke has been working on this chemistry application. He presented on it before and it'll be nice to kind of like talk about it a little bit more, but he has this, and correct me if I'm wrong, Luke, but there's like very... gross level is you have, you know, like a games application that's running, it generates some data, then that data gets consumed by some Python script. That Python script is written in Qiskit, it generates a bunch of circuits, runs those circuits, and then gets results back.

I'm going to stop here and maybe Luke and like fill in the gaps so that we can, before that, sorry, before that, my general idea is maybe if we start from the application and sort of like walk down an entire software stack might be a good idea. Lou, do you want to chime in and kind of give a little bit of a more detailed view of the application?

**[23:22-24:56] Bertels, Luke:**

Yeah, sure. So I think that is, I mean, I think you did a pretty good job describing it. So the idea is that we have a super molecular complex. We have some classical code that then takes that, decomposes it into a bunch of digestible quantum pieces. Those pieces are then fed to a circuit constructor that will build a VQE onsats for each of the individual complexes or each of the individual monomers and dimers coming from that complex decomposition. We would want to have some quantum orchestrator that would then schedule, execute those circuits at the level of a VQE.

You could then see there being some feedback where you're... doing parameter optimizations of those circuits in a hybrid quantum classical way, or if you were looking at something that was a little more new, that could be like a more of a single shot quantum sampling approach, which is what we've looked at more recently. And then going from there, you have kind of a reconstructor that gets energies and properties from, or yeah, that takes all those individual monomer and dimer energies, reconstructs energies and properties from those. This is kind of the overall. Workflow.

**[24:57-25:08] Shehata, Amir:**

Is there a loopback loop where after you get results, you feedback into the game side so that you can do further? Right, we go through the loop at that.

### 25:00-30:00

**[25:08-25:34] Bertels, Luke:**

Potentially, there could be, because... Yeah, so you could envision a loop back where then you use the quantum results to build the electrostatic potential, which defines a new embedding and kind of solve that whole big loop self-consistently. There's also... I guess a feedback loop at the at the DQE level.

**[25:37-25:59] Shehata, Amir:**

Yeah. The other question I have there, and that sort of asking that to understand whether we need to, you know, do co-location of quantum and classical. In the tests that you run, how long does the classical portion take and how long does the quantum portion take? For the problem size that you're working on.

**[25:59-26:18] Bertels, Luke:**

Sure, so for the problem sizes that we were looking at, and these were more proof of concept, the... Classical side was. I don't know, couple seconds. Um... Yeah, quantum times.

**[26:21-26:32] Bertels, Luke:**

I mean, the work we did with quantum brilliance were on the order of days. But with a faster processor, I think it would be... Probably hours.

**[26:35-26:44] Shehata, Amir:**

Or. So, like, did we try on IQM? I, I ran it on IQM, but I never ran like a full-size.

**[26:45-26:52] Bertels, Luke:**

Yeah, I don't know that we tried on IQM, or I haven't tried on IQM, like a full-size example. So that might be worth doing.

**[26:50-26:51] Shehata, Amir:**

Okay.

**[26:55-26:57] Bertels, Luke:**

There.

**[26:56-26:58] Shehata, Amir:**

So 2 seconds versus hours, okay.

**[26:58-27:18] Bertels, Luke:**

Yeah. There's another application that I've come across recently. which is based on this neural network quantum state approach. So have you met Manas yet, Amir? He recently got hired. Yeah.

**[27:15-27:17] Shehata, Amir:**

Yeah, yeah, it was an.

**[27:18-27:52] Bertels, Luke:**

So he has an interesting paper where they're using the quantum hardware to essentially do Markov chain sampling. And that's occurring at the single shot level. So you'll take a shot, use that shot to update some probabilities. So that, I don't know, when we were talking about things that require like low latency. Between the classical and the quantum, that was something that kind of... Piqued my interest, so it might be worth pulling him into this in some way.

**[27:54-27:59] Shehata, Amir:**

Does he have something that's actually running, like actual code that we can play with, or is it just paper?

**[28:00-28:14] Bertels, Luke:**

I think he has actual code. I don't remember where he ran his examples. It might have been with IBM. But yeah, as I recall from the paper, I think he did run some calculations on hardware.

**[28:16-29:01] Shehata, Amir:**

Okay, so we have, okay, we have two approaches here. We have Luke's application, which is more of a workflow-based application, so that that might lend itself well to setting up an experiment where we have used Turcries, for example, to formulate the games plus the... the Python scripts run, right? But even the Python scripts themselves are VQE. So they have to run, like they have some optimization loop there, so they run circuit and get results optimized and so on. But Luke, isn't there also the potential? actually running multiple optimization or multiple circuits in parallel.

**[29:04-29:16] Bertels, Luke:**

Yeah, so once you break up that super molecular complex into monomer and dimer systems, in principle, those can all be run in parallel.

**[29:18-29:21] Shehata, Amir:**

So you can have multiple BQEs running in parallel, right?

**[29:18-29:19] Bertels, Luke:**

Yeah.

**[29:23-29:24] Shehata, Amir:**

Sorry, I got someone else.

**[29:32-29:34] Shehata, Amir:**

Did somebody want to add something? Sorry.

**[29:38-30:25] Shehata, Amir:**

So my proposal, just to kind of throw it out there, see what you guys think, what if we take that application and sort of like we did that, so to be fully transparent, that work that Luke is talking about is part of an LDRD project, like a lab directed research and development project that we have here. So we have that The script running, sorry, the. the like the application there, but we just run sort of the circuit piece of it. So like the Python piece of it, we don't run the games.

The games is pre-run, we get the data and then we just sort of just run the Python scripts that, you know, turn the data and we only have, I don't think we did the. parallel VQEs, the parallel breakdown, right? Luke, we just did one.

### 30:00-35:00

**[30:27-30:32] Bertels, Luke:**

So we did it with the quantum brilliance, but not on the not on IQM.

**[30:33-31:11] Shehata, Amir:**

Okay. So we can potentially have the multiple parallel runs. Um... configured or run. So would it make sense to take that application as our first use case and actually just like... Set up something that runs from A to Z on our test bed using, since Tercrise is here, we can set up Tercrise, we can set up how we can integrate with our level, how like figure out how we can actually run the like the different operations in parallel. Um...

**[31:14-31:34] Shehata, Amir:**

And whether it is all, like, is it represented, I guess one question I would have, is the parallel operations represented at the tech rest level, or is it represented some at the lower level? Like you allocate the resources and then the application that runs then decides I'm going to run these operations in parallel.

**[31:40-32:26] Cambridge SuiteB GREAT OUSE (6):**

It's probably worth comparing them, but we do have the capability to encode that in the take rest graph. We have like support for parallelism and for loops, so... I think it's worth, yeah, comparing it and then comparing it to just a regular Python script as well, or thanks a lot, and there's also scheduler specific stuff, so when you when you say you run parallel stuff, this could also mean that you're submitting. programs in parallel to Slurm, but you could also be submitting a batch job script to Slurm, which then could have a different effect. for the resource allocation.

So there are multiple ways, I think, but it's definitely worth comparing the approaches.

**[32:27-33:31] Shehata, Amir:**

Yeah, I think that would be a good first step to look at the different approaches of how to manage parallel applications at the workflow level, at the actual just like you. And we have like this simulator that we carry, or not simulator, this Slurm cluster that is. just a Slurm virtual cluster, the Docker containers, and it has multiple QPUs in there. And you can potentially just run across the different QPUs, so you can test that experiment where you run at the workflow, you generate parallel calls into Slurm that walk in as Slurm jobs, and then those get run. on the different, you know, resources that are available.

And then we can have another one where you just submit one job that goes through Slurm, and then it reserves multiple resources, and then the Slurm job itself is the one that parallelizes internally. So that will compare both different. Does that make sense?

**[33:32-34:42] Cambridge SuiteB GREAT OUSE (6):**

I think it's quite interesting because we haven't done things with parallel QPUs right now. We don't sequential QPUs, but not in parallel. So that'd be interesting for us as well. So maybe also, and I think this is something a lot of people are currently researching or publishing work on is There's been a lot of stuff like what are the type of parallelism that exist in quantum algorithms. Because when we talk about VQE, we already know that current times don't really, or the latency times don't really matter because that the end result of a VQE doesn't depend on the shots being run in very close proximity to each other.

The only thing that matters is that they're run on the same calibration if you want. But then people started exploring also other areas where this might not be true, where it might matter. I mean, I think there's a couple of taxonomy systems out there. So once we have our first initial example going, we might also want to expand this into other areas.

**[34:45-34:55] Shehata, Amir:**

Yeah, it makes sense, I think. Okay, but like starting with the simpler stuff like the action, because the code that we have right now

**[34:52-34:53] Cambridge SuiteB GREAT OUSE (6):**

Yeah, I do.

**[34:55-35:11] Shehata, Amir:**

exists and we're able to like generate different types of parallelism, whether at the workflow level or at the application level. So maybe we can start with those two and then move more towards. What you were mentioning?

### 35:00-40:00

**[35:09-35:26] Cambridge SuiteB GREAT OUSE (6):**

Yeah, yeah. We also, I just wanted to mention, we also have like experiments where we had VQEs basically doing exactly that through tier price. But again, not like multiple QPUs, always single QPU, not time critical.

**[35:26-35:26] Shehata, Amir:**

Mm.

**[35:34-36:30] Shehata, Amir:**

So it feels to me that this approach, the top-down approach, makes a lot more sense right now until we figure out what the workflow is. And hopefully we can see a little bit more details what SpaceRT is trying to accomplish, like actual code and run some runs that we can actually understand a little bit more and then we can see how we can integrate that with the top down approach. So we're coming bottom up and top down and meeting sort of in the middle. But and we can do in parallel also because AMD is already working on the space RT level.

We can start looking at the application level and deploying some sort of like the Lux application. and trying to get that working from the top down, and then we can see, experiment, and understand a little bit more how we can meet in the middle. Does that make sense, or?

**[36:32-36:33] Tang, Xulong:**

No, the the.

**[36:33-36:36] Eckert, Yasuko:**

Sorry, I joined late, but, ohh, Shilong, were you going to say something?

**[36:37-36:39] Tang, Xulong:**

Yeah, go ahead, yes, once we finish.

**[36:39-37:22] Eckert, Yasuko:**

Yeah, and then sorry I missed last week's call too, but it sounds like the approach makes sense. Just wanted to get a little bit more information. I'm sure you already covered this, so sorry for the repeat, but... So it sounds like the target application is VQE. VQE by itself is just quantum. So is there like hybrid application in mind? Something that uses that runs on the classical side? aside from, you know, what's the feedback needed on the BQE side.

**[37:23-38:54] Shehata, Amir:**

So the application that we were just discussing has a classical side right now, which is a games, like a games application that runs on the classical. But when like Luke was just mentioning the different runtime is the classical would just run for like 2 seconds and then the. Quantum would run for longer. Like they ran it, I think, on the two qubit devices that we have here at Quantum Brilliance, and they ran for days. He was saying we haven't run on IQM yet, so we don't know how long the time is.

But even for VQE, there is a classical component to it, because you are doing an optimization loop, so you're running circuits, then getting bad results. And the The other part also is that the application you can actually parallelize the solution, so if you can have different components that you're solving for and they can run in parallel, so that's why we're thinking if we use that as a like as a trial run to set up our entire sort of top to bottom software stack. We can have the workflow manager orchestrate the classical and the quantum.

You know, the classical runs, then the quantum can actually also reserve some classical components because it needs to run on some classical resource to do the optimization. And you can actually use multiple QPUs because you're running multiple. solution at the same or trying to solve multiple components at the same time.

**[38:56-39:07] Eckert, Yasuko:**

But the classical side, is it just the optimization needed for BQE or is it there? Okay, so yeah, so I think that's a great starting point,

**[39:01-39:05] Shehata, Amir:**

Yeah, yeah. Yeah, I just.

**[39:07-39:14] Eckert, Yasuko:**

but I think expanding that to a full application later on might be a good next

**[39:13-39:16] Shehata, Amir:**

Yeah, yeah, could be 100%, you know, I'm just thinking.

**[39:14-39:16] Eckert, Yasuko:**

step. Yeah, but for that.

**[39:15-39:16] Cambridge SuiteB GREAT OUSE (6):**

Thank you.

**[39:17-39:18] Shehata, Amir:**

Seven.

**[39:18-39:22] Cambridge SuiteB GREAT OUSE (6):**

Yeah, okay, then I see. But this is basically exactly the thing I

**[39:19-39:19] Shehata, Amir:**

Not offensive though.

**[39:22-40:01] Cambridge SuiteB GREAT OUSE (6):**

wanted to point out as well, that the problem currently is that it's very hard to find applications where we either not have a classical trivial thing, so we either have to come up with something classically that is just pretending to be difficult, or we immediately get to the limits of quantum hardware. So it's very hard to find an application that currently does both quantum interesting and classically interesting. This is something that is very difficult to do at the moment. So doing something classically simple and in the worst case, pretending that it is difficult, I think is currently the easiest approach.

### 40:00-45:00

**[40:03-40:10] Eckert, Yasuko:**

Yeah, I, I, I hear you. That's the current state, um, and...

**[40:09-40:33] Shehata, Amir:**

I think also the parallelization, sorry to cut in, the parallelization and the loop, because Luke was saying that there is a potential to actually have a wider loop where, you know, you're on your games, you get some data, you're on your VQE and you get results and then you loop it back and run games again. So that bigger loop if we implement it, I don't, we haven't done that at least, right?

**[40:36-40:39] Cambridge SuiteB GREAT OUSE (6):**

But it would still be classically trivial, right?

**[40:36-40:38] Bertels, Luke:**

Yeah, right, so the...

**[40:39-40:40] Shehata, Amir:**

Right, right, yeah.

**[40:44-40:45] Eckert, Yasuko:**

So, that...

**[40:45-40:46] Shehata, Amir:**

Sorry, what?

**[40:46-40:54] Eckert, Yasuko:**

Oh, parallelization, but you only have one QPU.

**[40:55-40:56] Shehata, Amir:**

Ohh.

**[40:55-40:57] Eckert, Yasuko:**

At Pathfinder, right?

**[40:58-41:09] Shehata, Amir:**

Right. We're thinking of emulating the other QPUs. You'll have some sort of... simulators in the back end that are emulating the different QPUs.

**[41:10-41:12] Eckert, Yasuko:**

Got it, got it, and now running.

**[41:11-41:12] Shehata, Amir:**

Yeah, and.

**[41:13-41:19] Eckert, Yasuko:**

Sorry, just one more quick for running the entire loop on the test bed.

**[41:20-41:20] Shehata, Amir:**

Right.

**[41:23-41:39] Shehata, Amir:**

So we start, I was thinking if we need to kind of think on it a little bit to find, like to break down the actual steps that we need to go through, but it seems we need, we have this learn Docker environment. I can share it with you guys right now.

**[41:52-41:54] Shehata, Amir:**

Speak up, Srikar, if you have something to say.

**[41:56-42:01] Srikar Chundury:**

Well, I, yes, I don't think it's super related, so I just wanted to add it as a chat, yeah.

**[42:01-42:03] Shehata, Amir:**

What do you want to say?

**[42:04-43:01] Srikar Chundury:**

No, I was talking about why are we thinking about just parallelism at the Slurm level? Because at the end we are bottlenecked by how to share the QPUs, right? So the different levels of runtime that you were mentioning, I was thinking should have information about everything in the stack, not just Slurm. And also how granular it is. So for instance, certain hardware does not do qubit sharing across, like for different jobs on the same QPU versus some do. There are like, what do you call, errors that come with jobs that share the same QPU. and other parameters that we want to consider and so on.

So yeah, so I just wanted to think about can the runtime handle that information as well? And if so, is it going to be at an interface level or at the implementation level?

**[43:02-43:37] Shehata, Amir:**

That's a good point. So from the sharing of the QPU, there was a couple of, like you mentioned, there's time sharing and space sharing. So space sharing is the idea. You have one application or multiple applications running on the different parts of the QPU. And time sharing is more of, you know, you, well, yeah, you have a single QPU, but you. multitask among the different circuits that need to run. Is that what you're meaning?

**[43:37-43:41] Srikar Chundury:**

Yes, I was thinking, would the runtime take care of this or should it take care of it?

**[43:43-44:51] Shehata, Amir:**

At some level, I think it will need to do that. And we have some work that with the scheduling and admission control that we did that allows only addresses that I'm sharing bit. Because right now we have this layer of admission control. multiple jobs can come and request access to the same QPU. And then the QPU has a mechanism or the admission control layer that we wrote can look at the QPU capacity and see if there's enough capacity to run the next job that's coming.

And the way that happens is the job that's being submitted defines a bunch of different metadata, like the maximum number of circuits that will ever run, how the maximum number of qubits, that maximum depth, things like that, and we use them some sort of estimation, resource estimation, to figure out how much capacity is wired on the QPU and then we can therefore accept multiple jobs at the same time. And then you have a scheduler in the back

**[44:50-44:50] Srikar Chundury:**

Not sure.

**[44:51-44:57] Shehata, Amir:**

end that then schedules the tasks that are coming up from each one of the hybrid jobs.

**[44:58-45:03] Srikar Chundury:**

Gotcha. Thank you. So the slum level knows how much it can parallelize.

### 45:00-50:00

**[45:04-45:07] Shehata, Amir:**

Right, yeah, by communicating with the QPU.

**[45:04-45:08] Srikar Chundury:**

OK, perfect. Right, yeah, OK.

**[45:09-45:23] Shehata, Amir:**

So can that that actually is another layer because you can have parallelism can look like you're using multiple QPUs or it can look like you have multiple jobs using the same QPU.

**[45:24-45:26] Srikar Chundury:**

Right, okay, yeah, watch.

**[45:31-45:36] Shehata, Amir:**

Um... Sorry, Hatim, did you want to say something?

**[45:40-46:21] Abdur Rahman Hatim:**

Yeah, so I was just looking at this paper about sharing QPUs in which even when we're sharing, trying to share a QPU between multiple workloads, there are different ways in which we can do this either by temporarily sharing it by interleaving shots or In the context of resource, or we can go a layer above up to the workflow level, or either through circuit knitting as well, so that again comes back to the runtime or the application level interface that we're trying to. Look at.

**[46:23-46:50] Shehata, Amir:**

Yeah. So it seems to me, I think it would be the best approach is to set up some prototype that we can actually experiment, and then those questions become more tangible, and then we can actually start answering them instead of theoretically thinking about it. So that's, I'm a more hands-on person, so I would. Much rather. like get into the weeds, deploy something and try it out.

**[46:50-47:38] Cambridge SuiteB GREAT OUSE (6):**

And also, so I completely agree, and I think this, if we start thinking about this like runtime sharing or QPO sharing, I think this only distracts for now from what we're trying to achieve, because it's first not clear where it will happen yet. And also, at least on the current hardware, at least from, so this is stuff I did in my PhD. Sharing is basically only interesting from the point of the scheduler, not from the application, because applications that use resource sharing tend to do a lot worse than if you're not doing it.

So I think it's quite good if you focus on something that Okay, we can investigate and once we have the system running, we can try things out, but I wouldn't make it a first priority.

**[47:41-47:52] Shehata, Amir:**

Fair. I think I agree like having setting up something that runs first and then afterwards introducing sharing might be the right like workflow.

**[47:55-48:09] Cambridge SuiteB GREAT OUSE (6):**

So you've got this slurm setup example repo. Does it make sense to make like just a new repo where we can start putting in a few different alternative ways of running the same experiment that we can compare? I don't know.

**[48:08-48:14] Shehata, Amir:**

Um, if you... When you say different, like a different cluster or...

**[48:15-48:54] Cambridge SuiteB GREAT OUSE (6):**

So like just have a repo that has some like PyTest benchmark or something similar to, I don't know what a good benchmarking framework here would be or some test framework where you can run a whole bunch of different methods of doing this same experiment to compare against each other. So if we want to like actually get numbers on. the time for different steps or how parallelizable things are. Just a suggestion. If we want to get our hands dirty, making a repo and trying to see how the code actually integrates together and then benchmarking it together. Does that make sense?

**[48:54-49:06] Shehata, Amir:**

There, I, I have no problem with creating another and setting things up. I like, I think what I'm leaning towards is because we want to use the test bed that is local at Oak Ridge,

**[49:05-49:05] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[49:06-49:40] Shehata, Amir:**

where like we're deploying the framework that we're developing on that test bed, so we need to kind of work. work within it to make sure that we can actually deploy it so that because it's going to be deployed and then it's going to be other applications like this are going to be writing their applications and using that framework. So if we can sort of try to fit that within that framework, that would be a nice thing to do. That's learn cluster that I shared with you. Basically, maybe I can share quickly what it looks like.

**[49:48-49:50] Shehata, Amir:**

Let me share my screen.

**[49:58-50:00] Shehata, Amir:**

So I started this dashboard thing here.

### 50:00-55:00

**[50:03-53:05] Shehata, Amir:**

So this is what it kind of looks like internally, right? So you have two partitions. You have a normal partition and you have the services partition. The normal partition is composed of eight nodes. Those are just classical nodes that you can allocate. And then the classical partition has like a node for like a back end called a fake IQM. It's just like a simulated back end that doesn't do anything. Just kind of like to experiment with the interface. Then we have an IQM back end node which talks to the IQM directly.

So if you're running locally here. that you can actually put the API keys and you can talk to the IQM using the IQM client directly. Then you have this shim head here. This shim is the, it's not backed by the IQM client, but backed by QRMI and QDMI. So instead of talking to the IQM. directly via their IQM client, you're talking to the IQM player via the QDMI, QRMI. And then NWQ SIM is just, it's like a simulator. And it has three nodes, NWQ SIM, worker one, two, and three. And those guys are just.

They're there to, like, so that you, if you're on a really large circuit, it can actually run across all four nodes. But it's a Docker container, so I don't know how large it was, so that you can run anyway, right? So what I was hoping is what we can do is on the Slurm controller node, we can configure Turk rice, right? And Turk rice can run here. And then that can allow us to allocate. or run some like application that can allocate the nodes from the normal partition and can also talk and get some services, QFW services. We still have to figure out how we can integrate that in.

And then. for parallelization, you can technically have multiple circuits or multiple, like if we break down the loops application into multiple components that need to be solved independently, they can use different nodes for the. for the VQE or the actual running the VQE script, and then they can talk to the same IQM, or they can talk to like the IQM and then in the VQ sim. So some of them can be run directly on the IQM system, and some can run on the state vector simulator, which is in the VQ sim. So that's the kind of idea that I had with you guys think.

**[53:08-53:09] Cambridge SuiteB GREAT OUSE (6):**

Yeah, and.

**[53:13-53:15] Shehata, Amir:**

So throwing it back in your core, John, though, like,

**[53:13-53:14] Cambridge SuiteB GREAT OUSE (6):**

Question.

**[53:15-53:20] Shehata, Amir:**

what are you thinking about the other repo and what do you want to pull in?

**[53:23-53:53] Cambridge SuiteB GREAT OUSE (6):**

I think this was more like a benchmarks eventually, but I guess I'm just thinking about the experiments we did so far where we have like a repo just where we put the code to make it so that people can reproduce it. But yeah, I guess what I was thinking was just how do we start? Really, and I think this is a good place. I just, when we want to actually provision onto the test bed as well, I was just maybe to ask you really about how that should look.

**[53:54-54:56] Shehata, Amir:**

So this is supposed to reflect a little bit of what the real test bed will look like, right? Because in the real test bed, we'll have some nodes, so we'll have like 4 CPU nodes and one AMD GPU node, right? The AMD GPU node is beefy, it has like 8 GPUs. And then the services are going to be running. So the IQM service here, that node is actually going to be the like the Dell server that is in the IQM rack, right? And then technically we can have NWQ SIM running also on. like on a couple of nodes just to have a simulator services.

So in a sense, this kind of looks like what the real test bed is going to be. And... What I'm hoping we can do is if we deploy and use that as our test environment, then it becomes easier to just sort of mirror that on the real test bed once

**[54:56-54:56] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[54:56-55:19] Shehata, Amir:**

it's available. Because I got like, I guess Continuum was part of QSC, so you guys might have access to the test bed already, and AMD is part of QSC. But there might be others that are not part of QSC, and then having that as an intermediate place where we can test things and exchange ideas might be a good way of. Yeah, collaborating.

### 55:00-60:00

**[55:21-55:21] Cambridge SuiteB GREAT OUSE (6):**

Yeah, makes sense.

**[55:28-55:30] Shehata, Amir:**

Okay, so what should our next steps be?

**[55:32-56:07] Cambridge SuiteB GREAT OUSE (6):**

I guess from our point, this would probably like sort out the access stuff and probably have a look at the implementation of Luke's code. And then we can start building our stuff, and yeah, there's a few things we need to do to support. I think we don't currently have QMI support, so it's probably how we would want to interact with that. Yeah, but I think that's pretty trivial, trivial, but yeah, decides to.

**[56:07-56:19] Shehata, Amir:**

So technically, you're not going to be dealing with QRMI, QDMI directly or going to be dealing with our service. So that QPM here, right, that those blue or golden things, those are the ones that you're going to

**[56:16-56:16] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[56:19-56:24] Shehata, Amir:**

be dealing with. You're not going to be dealing with the QRMI, QDMI is kind of hidden behind it.

**[56:25-56:30] Cambridge SuiteB GREAT OUSE (6):**

OK, yeah, but I think that should be part. Either way, we just need to have something to interrupt with that.

**[56:31-57:10] Shehata, Amir:**

So how about this? Maybe next time we'll meet, which is going to be in two weeks, because next week is QC. But what about Luke can give us a rundown of the code, how we can deploy it. We create a repository with his code so that we can actually pull it and run it and test with it independently. And then the next step would be you guys can take on this or like try to run it and integrate their price in one of the like as a separate Docker layer or whatever so that they can work within that environment. Does that make sense?

**[57:09-57:11] Cambridge SuiteB GREAT OUSE (6):**

Yeah, yeah.

**[57:13-57:28] Eckert, Yasuko:**

I think for my education, a high level overview of how the application workflow might work, works would be really educational and then I can map that out to what Amir is showing here.

**[57:30-57:34] Shehata, Amir:**

That's a good idea. So I think if we get Luke and he's typing, do you want to say something, Luke?

**[57:36-57:41] Bertels, Luke:**

Yeah, we already have that code in an open QSC repo somewhere, don't we?

**[57:41-57:44] Shehata, Amir:**

Yeah, I think we do. I can add everybody here on it.

**[57:48-57:51] Shehata, Amir:**

You okay with us adding people, Luke?

**[57:50-57:52] Bertels, Luke:**

Yep. Yeah, that's good.

**[58:21-58:21] Shehata, Amir:**

Um...

**[58:21-58:22] Cambridge SuiteB GREAT OUSE (6):**

The.

**[58:24-58:35] Shehata, Amir:**

So, what can I add? I'll add you guys, I'm not going to waste time on this. I'll add everybody in this group on this chemistry application,

**[58:32-58:32] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[58:35-58:42] Shehata, Amir:**

then you'll have it, you'll have access to it, and then you can basically take it and run it on your system and learn more about it.

**[58:43-58:44] Cambridge SuiteB GREAT OUSE (6):**

Sounds good. Thank you very much.

**[58:46-58:49] Shehata, Amir:**

Sounds good. So next time maybe Luke and are you okay

**[58:46-58:47] Abdur Rahman Hatim:**

Thank you.

**[58:49-59:01] Shehata, Amir:**

with that Luke to give us an overview like of the application again so that we refresh our memory, maybe walk through the code and then we can like have that for the next meeting and then we can.

**[59:01-59:03] Bertels, Luke:**

Sure, that's in roughly 2 weeks.

**[59:03-59:04] Shehata, Amir:**

Two weeks, yeah.

**[59:04-59:06] Bertels, Luke:**

Okay, yeah, I should be should be game for that.

**[59:08-59:10] Shehata, Amir:**

Does that sound good with everybody?

**[59:11-59:11] Cambridge SuiteB GREAT OUSE (6):**

Yeah.

**[59:12-59:13] Shehata, Amir:**

All right, awesome.

**[59:12-59:13] Abdur Rahman Hatim:**

Yep.

**[59:14-59:19] Shehata, Amir:**

Okay, cool. Thank you for your time and effort.

**[59:17-59:18] Tang, Xulong:**

Thank you.

**[59:20-59:20] Srikar Chundury:**

Thank you.

**[59:20-59:22] Shehata, Amir:**

Bye, take care, guys. Yeah.

**[59:21-59:21] Tang, Xulong:**

Thank you.

**[59:22-59:24] Cambridge SuiteB GREAT OUSE (6):**

If you can't, bye bye.

**[59:24-59:24] Shehata, Amir:**

The.

**[59:25-59:25] Tang, Xulong:**

Okay.
