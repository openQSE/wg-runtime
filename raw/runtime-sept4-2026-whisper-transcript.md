# Transcript

Model: `base.en`

[00:00:00 - 00:00:07] or the particular work that I look at, that might be important.
[00:00:07 - 00:00:13] So one of the key observations is that there is no standard device runtime and a task interface
[00:00:13 - 00:00:17] that is shared across quantum and HPC stacks.
[00:00:17 - 00:00:25] If there exists, it will be really nice to formalize it within this group as well,
[00:00:25 - 00:00:31] And not just in the perspective of runtime for something like
[00:00:31 - 00:00:36] Slurm or Jot Dispatch, but runtime that targeted the device
[00:00:36 - 00:00:37] side to things.
[00:00:37 - 00:00:40] And I'll give an example of what that means when we talk
[00:00:40 - 00:00:43] about classical, which is essentially this.
[00:00:43 - 00:00:47] So if you're not familiar with Kudos for runtime
[00:00:47 - 00:00:54] or our Rockam hit runtime, some of the components are there
[00:00:54 - 00:00:59] is a very stream centric model to how we build applications
[00:00:59 - 00:01:02] and kernels on these GPUs.
[00:01:02 - 00:01:04] And it's a really good example to look at,
[00:01:04 - 00:01:07] because when we talk about heterogeneous quantum systems,
[00:01:07 - 00:01:09] GPUs are a big part of it.
[00:01:09 - 00:01:12] And there might be CPUs or FPGAs and other pieces of it,
[00:01:12 - 00:01:14] too, which we're internally exploring,
[00:01:14 - 00:01:17] because AMD has a portfolio that spans across a bunch
[00:01:17 - 00:01:20] of these products.
[00:01:20 - 00:01:24] And the idea is that in CUDA runtime,
[00:01:24 - 00:01:27] which I won't go over the right side,
[00:01:27 - 00:01:31] which it basically mirrors whatever CUDA provides,
[00:01:31 - 00:01:35] but there are, you manually manage these contexts,
[00:01:36 - 00:01:39] which are sort of device context
[00:01:39 - 00:01:40] that gives you information
[00:01:40 - 00:01:42] of what the underlying hardware provides.
[00:01:42 - 00:01:46] And these are resources, the physical resources
[00:01:46 - 00:01:47] that the hardware provides.
[00:01:47 - 00:01:50] So it will be things like number of compute units available
[00:01:50 - 00:01:54] within the hardware number, the amount of scratchpad memory
[00:01:54 - 00:01:56] available, the amount of HPM memory available.
[00:01:56 - 00:01:59] So all sorts of different information
[00:01:59 - 00:02:03] that lets you program for that specific system.
[00:02:03 - 00:02:06] And the actual programming model is that you
[00:02:06 - 00:02:09] split between something called the shader code
[00:02:09 - 00:02:13] or often known as a kernel, which executes on the device
[00:02:13 - 00:02:15] itself, and then a host code that
[00:02:15 - 00:02:19] is responsible for launching that work on to the device.
[00:02:19 - 00:02:22] and that host code typically resides in the CPU.
[00:02:22 - 00:02:26] So CPU is responsible for launching the work onto the GPU
[00:02:26 - 00:02:29] and the GPU is responsible for consuming that work.
[00:02:29 - 00:02:32] Although that's changing as we go more and more towards
[00:02:32 - 00:02:37] LLM, there have been entirely device-centric runtimes,
[00:02:39 - 00:02:40] which I'll talk about in a second,
[00:02:40 - 00:02:44] where the device itself launches the work on itself
[00:02:44 - 00:02:47] or on other devices, which is kind of cool to think about.
[00:02:47 - 00:02:54] And all of that is done to reduce latency, which I'll also get to in a second, because
[00:02:54 - 00:03:00] latency is super important in inference workloads for LLMs as well.
[00:03:00 - 00:03:07] So it's very nice, overlapping what latency requirements are for quantum and also where
[00:03:07 - 00:03:14] the industry is turning towards for AI, although not in the same way.
[00:03:14 - 00:03:19] There are very strict requirements in quantum, whereas there's loose performance and power
[00:03:19 - 00:03:22] requirements on the AI side.
[00:03:22 - 00:03:26] The main thing that I'd like to discuss is this API.
[00:03:26 - 00:03:28] It kind of tells you a lot.
[00:03:28 - 00:03:35] It's the notorious triple chevron API, and the triple chevron API within CUDA kernels
[00:03:35 - 00:03:37] is how you launch the kernel.
[00:03:37 - 00:03:42] So this is essentially encompasses a lot of the pieces of runtime itself.
[00:03:42 - 00:03:45] There are a few different parts of this.
[00:03:45 - 00:03:52] There's something called Grid and Block, which are essentially like your logical work items
[00:03:52 - 00:03:58] that you're constructing that will be eventually launched onto the physical hardware.
[00:03:58 - 00:04:04] And this has allowed CUDA to be actually very successful because after generation they have
[00:04:04 - 00:04:07] just added more compute resources onto the GPUs.
[00:04:07 - 00:04:12] The GPUs have evolved quite a lot, but one way that they have involved is they support
[00:04:12 - 00:04:15] more and more and more and more computer throughput.
[00:04:15 - 00:04:22] But the programming model was such that you always targeted these logical entities called
[00:04:22 - 00:04:24] grids and thread blocks.
[00:04:24 - 00:04:29] And the hardware scheduler then decides how to place these logical software entities onto
[00:04:29 - 00:04:32] the physical compute units of the GPU.
[00:04:32 - 00:04:36] So that allowed Kuda to scale really well because software was written for these logical
[00:04:36 - 00:04:40] entities, it was always separate from what happened on the hardware side. The hardware
[00:04:40 - 00:04:45] side got more and more, and as long as you had more of these logical stuff to schedule
[00:04:45 - 00:04:50] onto more of the physical compute units, you always were keeping the GPU busy and you were
[00:04:50 - 00:04:56] always keeping the resource busy, and your code just worked from generation to generation.
[00:04:56 - 00:04:59] That's changing quite a lot, but that's a different story.
[00:05:00 - 00:05:05] So the first piece is like your logical work items is how I call it.
[00:05:05 - 00:05:15] The second piece is the amount of scratchpad memory you're giving to each of the individual logical work item block.
[00:05:15 - 00:05:20] So this is limited by how much physical shared memory you have.
[00:05:20 - 00:05:23] Scratchpad memory, LDS, they're all synonymous.
[00:05:23 - 00:05:29] It's basically memory that is really close to the compute and typically used to store
[00:05:29 - 00:05:39] tiles of gems for reuse. So you'll store things that you will use more often in these
[00:05:39 - 00:05:43] kernels and these applications close to the core itself where they're going to be computed
[00:05:43 - 00:05:50] on. And this is a software controlled thing. That's why sometimes the programmer specifies
[00:05:50 - 00:05:56] here, sometimes not always, how much scratchpad memory is required for each one of my thread
[00:05:56 - 00:05:59] which is a logical work item.
[00:05:59 - 00:06:02] And then the last bit, which is super important,
[00:06:02 - 00:06:06] this is the crux of essentially all of the runtime component,
[00:06:06 - 00:06:08] is called the stream.
[00:06:08 - 00:06:11] And stream are these contexts.
[00:06:11 - 00:06:16] That is how you establish concurrency within GPU runtimes.
[00:06:16 - 00:06:19] So it's basically saying, I know that I
[00:06:19 - 00:06:22] can create a task graph of my application.
[00:06:22 - 00:06:25] And let's say my task graph looks something like this.
[00:06:25 - 00:06:29] I have a pre-processing step and that is represented as a node.
[00:06:29 - 00:06:35] And that pre-processing step launches some work that is represented as like another node.
[00:06:35 - 00:06:41] And then that work is going to create, not create, but is going to move on to an X set
[00:06:41 - 00:06:42] of work.
[00:06:42 - 00:06:46] And that might move on to two independent sets of work.
[00:06:46 - 00:06:49] And then I have to join them together to do some post-processing.
[00:06:49 - 00:06:52] And then I have basically finished my application workload.
[00:06:52 - 00:06:59] So this task graph that we are very naturally kind of used to envisioning in many of the
[00:06:59 - 00:07:07] scientific applications and HBCs and AI and all types of workloads that we built.
[00:07:07 - 00:07:14] This can be done at a very high level too by the way, but think of it as some pre-processing,
[00:07:14 - 00:07:19] some task happening, some working happening, and then some joining happening and then eventually
[00:07:19 - 00:07:26] the ending. This graph is was always or for a really long time represented using these
[00:07:26 - 00:07:36] things called streams. So streams were essentially one sequential path of this graph. And what
[00:07:36 - 00:07:43] you will say is I have this stream a and it's going to have these these nodes on it, these
[00:07:43 - 00:07:49] kernels that I will run and then I will have a stream B somewhere here that will
[00:07:49 - 00:07:53] have this node on it and I will have mechanisms to synchronize the streams
[00:07:53 - 00:07:59] when I need to create essentially this task graph model. So CUDA was very much
[00:07:59 - 00:08:03] a stream centric model. Concurrency was expressed through these streams,
[00:08:03 - 00:08:09] kernels were run on these streams, mem copies were done between the host
[00:08:09 - 00:08:14] and device through the streams. Events are another thing that was used to
[00:08:14 - 00:08:21] synchronize these streams, these little lines that have drawn where the graph
[00:08:21 - 00:08:31] forks or joins. And that's what the crux of CUDA's runtime model to express
[00:08:31 - 00:08:41] concurrency and express these graphs and tasks. Later they realized that we are just
[00:08:41 - 00:08:48] building task graphs and then they added something called the graph capture which is essentially
[00:08:48 - 00:08:52] a way to capture these streams into a graph and then once you build the entire graph you
[00:08:52 - 00:08:59] can launch the entire graph all at once instead of launching a kernel at a time and that allowed
[00:08:59 - 00:09:03] them to cut down a lot of latency of launching these kernels.
[00:09:03 - 00:09:09] So imagine a quantum workload where you have a ton of gates,
[00:09:09 - 00:09:12] like extreme amount of gates that you have to apply.
[00:09:12 - 00:09:15] And each one of them ended up being a kernel launch.
[00:09:15 - 00:09:18] And kernel launch was 10 microseconds.
[00:09:18 - 00:09:21] So you will have 10 microseconds times the number of launches
[00:09:21 - 00:09:23] that you have to do, let's say a million launches
[00:09:23 - 00:09:25] that you have to do.
[00:09:25 - 00:09:27] Most of your time is actually being spent just launching
[00:09:27 - 00:09:28] those kernels.
[00:09:28 - 00:09:34] So a lot of optimizations were done on how to reduce that launch latency and let that
[00:09:34 - 00:09:40] not be an artificial bottleneck and have the compute be the actual bottleneck.
[00:09:40 - 00:09:46] So that's what task graphs are building this graph capture using the streams or explicitly
[00:09:46 - 00:09:53] building this, representing this DAG and launching it all at once onto the GPU is all about.
[00:09:53 - 00:09:55] Thank you.
[00:10:00 - 00:10:08] Yeah, that's just an example.
[00:10:08 - 00:10:09] This is an example.
[00:10:09 - 00:10:16] So typically you could represent a gate with a thread within a kernel.
[00:10:16 - 00:10:20] So like one run work item within a kernel.
[00:10:20 - 00:10:22] There's a bunch of different ways to map this.
[00:10:22 - 00:10:28] And all depends on how you want to load balance your computation on top of the GPU threads.
[00:10:28 - 00:10:32] So GPUs are really, really, really massively parallel devices.
[00:10:32 - 00:10:39] And if you're launching one gate per one kernel, it's pretty bad implementation.
[00:10:39 - 00:10:51] You might want to do, paralyze it over qubits or some sort of finer granular object.
[00:10:51 - 00:10:57] Yeah.
[00:10:57 - 00:11:00] American correction or simulations.
[00:11:00 - 00:11:05] So if you're simulating a circuit onto a classical computer
[00:11:05 - 00:11:07] and how that maps onto it, that's
[00:11:07 - 00:11:09] kind of what you would do, too.
[00:11:09 - 00:11:13] That's what the contents are.
[00:11:13 - 00:11:15] But that's just an example to illustrate
[00:11:15 - 00:11:18] that there are cases where you're entirely
[00:11:18 - 00:11:20] kernel launch latency bound.
[00:11:20 - 00:11:23] And these graphs allows you to reduce that,
[00:11:23 - 00:11:26] because you can build the entire graph off and ahead of time.
[00:11:26 - 00:11:28] And when you can build it ahead of time,
[00:11:28 - 00:11:30] you can queue it all ahead of time
[00:11:30 - 00:11:36] and pay it largely and see only once, which is very powerful.
[00:11:36 - 00:11:39] And this applies to not just the one example,
[00:11:39 - 00:11:44] but a lot of examples in graph algorithms on GPUs,
[00:11:44 - 00:11:48] which is a very classical like HPC case,
[00:11:48 - 00:11:51] or even some of the AI workloads.
[00:11:51 - 00:12:02] And then the last piece that I'll talk about here on this slide is the synchronizing synchronization
[00:12:02 - 00:12:03] mechanisms.
[00:12:03 - 00:12:09] So I talked about it a little bit already where you have these like spaces, places in
[00:12:09 - 00:12:14] the graph where you want to fork or join the graphs of these points.
[00:12:14 - 00:12:19] CUDA gives explicit APIs to synchronize streams as well.
[00:12:19 - 00:12:24] So there's a way to synchronize a specific stream to a certain point in the host code.
[00:12:24 - 00:12:30] So the CPU is saying here I will stop for all kernels within the stream to be done before
[00:12:30 - 00:12:35] I make forward progress because my forward progress kernel depends on the results from
[00:12:35 - 00:12:36] the previous kernels.
[00:12:36 - 00:12:38] So I want to synchronize my stream.
[00:12:38 - 00:12:44] I can synchronize the entire device, which is basically saying all workloads in the GPUs
[00:12:44 - 00:12:47] must synchronize to this point.
[00:12:47 - 00:12:52] all everything running on the device must synchronize to this point before we can make
[00:12:52 - 00:12:59] or launch more work or make forward progress on the CPU side. There's events which are
[00:12:59 - 00:13:05] creating more fine-grained dependencies between streams and maybe devices even, multi-GPU programming.
[00:13:05 - 00:13:11] And then there could be APIs that allow you to synchronize within a GPU kernel as well.
[00:13:11 - 00:13:16] So not having to launch separate kernels or queue separate kernels onto a single stream
[00:13:16 - 00:13:22] and synchronize, but synchronize the various logical items, the thread blocks,
[00:13:23 - 00:13:29] using something called the grid.sync. So it's basically a fine-grained synchronization mechanism
[00:13:30 - 00:13:33] and there's within the kernel itself, so you never come back to the host.
[00:13:34 - 00:13:41] So the top ones are all host-driven, which is host here is the CPU, and the grid.sync is
[00:13:41 - 00:13:42] from the GPU itself.
[00:13:42 - 00:13:46] So GPU saying, like, I will synchronize all logical work
[00:13:46 - 00:13:48] items to this point before I make forward progress.
[00:14:01 - 00:14:02] Yeah.
[00:14:02 - 00:14:17] No, so the application, if you are building the application, what typically happens is
[00:14:17 - 00:14:23] the kernels themselves that are launched with this triple chevron notation are wrapped
[00:14:23 - 00:14:28] in a nice Python or C++ API.
[00:14:28 - 00:14:36] And that Python or C++ API looks something like, you could say, computation, like computation,
[00:14:36 - 00:14:41] I'll just write computation a brackets, right?
[00:14:41 - 00:14:43] Like it's something like this.
[00:14:43 - 00:14:45] And the application lives up here.
[00:14:45 - 00:14:50] And the application is saying comp a, comp b, comp c, comp d.
[00:14:50 - 00:14:56] And then maybe, maybe some API specifically in torch allows you to express streams as
[00:14:56 - 00:15:00] well because you might want to say comp A and B or it can be concur.
[00:15:00 - 00:15:07] but COM C and B needs to be sequential. So there are some, only some at a high level,
[00:15:07 - 00:15:14] like higher level synchronization dependencies being expressed between these operators within
[00:15:14 - 00:15:20] something like Torch which builds these like larger applications, but not to the level of
[00:15:20 - 00:15:26] something like GoodSync. And there are finer levels than GoodSync too. There's within a workgroup
[00:15:26 - 00:15:30] you can synchronize logical things called warps array fronts and other stuff.
[00:15:30 - 00:15:34] But at an application level, you're just building this thing.
[00:15:36 - 00:15:39] Yeah, there's another question.
[00:15:40 - 00:15:47] Yeah, just to clarify, because I'm also not sure if this is or if I understand correctly.
[00:15:47 - 00:15:52] So for my understanding, a user would build something using torch.
[00:15:52 - 00:15:58] So for this group, are we trying to do something more like torch?
[00:15:58 - 00:16:02] Or are we trying to build the basics that is below torch?
[00:16:02 - 00:16:09] Because I think this is very much overlapping with the already existing stuff in QLIMI slash QM
[00:16:09 - 00:16:15] I, where we try to unify about the devices and their capabilities and so on.
[00:16:15 - 00:16:20] So I'm still not sure where we draw the line, okay,
[00:16:21 - 00:16:25] which is the responsibility of the runtime.
[00:16:25 - 00:16:27] Are we trying to build the device runtime?
[00:16:27 - 00:16:30] Are we running across multiple devices
[00:16:30 - 00:16:32] on the application level?
[00:16:32 - 00:16:34] Or are we running stuff that is actually
[00:16:34 - 00:16:35] then running on the QPU?
[00:16:35 - 00:16:40] Because I think this is closer to where these streams are, right?
[00:16:41 - 00:16:43] Yeah, I think that's a fair assessment.
[00:16:43 - 00:16:46] So the stuff that I personally work on
[00:16:46 - 00:16:49] is closer to the device side runtime,
[00:16:49 - 00:16:53] which is the layer that allows you to build
[00:16:53 - 00:16:57] what's inside these comp A's and comp B's and whatever,
[00:16:57 - 00:16:59] not at the Torx level.
[00:16:59 - 00:17:01] You could construct something on top of that
[00:17:01 - 00:17:05] that builds the Torx side coupling together
[00:17:05 - 00:17:08] and then putting together the final application you need.
[00:17:08 - 00:17:10] And that's something I'm also looking at separately,
[00:17:10 - 00:17:14] but space RT in particular, the project that I work on,
[00:17:14 - 00:17:17] focuses on the device-side runtime
[00:17:17 - 00:17:21] for a bunch of devices,
[00:17:21 - 00:17:22] not actually, I shouldn't say all.
[00:17:22 - 00:17:24] It's doing that for GPUs,
[00:17:24 - 00:17:28] GPUs, NTUs, and AIEs, FPGAs,
[00:17:28 - 00:17:31] and also core computers.
[00:17:32 - 00:17:34] And then as a follow-up,
[00:17:34 - 00:17:39] this would then also assume that hardware vendors
[00:17:39 - 00:17:42] allow you to install the SRAM type on the devices, right?
[00:17:42 - 00:17:43] I mean, I'm not.
[00:17:43 - 00:17:44] Correct.
[00:17:44 - 00:17:44] Yes.
[00:17:44 - 00:17:48] So really averted anyone or any of the vendors
[00:17:48 - 00:17:50] gives you this level of access, right?
[00:17:50 - 00:17:53] Where you're very, OK, these are our controllers, right?
[00:17:53 - 00:17:56] We control what is running on our hardware,
[00:17:56 - 00:17:59] and we don't even give you this level of access usually, right?
[00:17:59 - 00:18:00] Yeah, precisely.
[00:18:00 - 00:18:03] So for a quantum, that's correct.
[00:18:03 - 00:18:07] But when we are talking about like coupling together,
[00:18:07 - 00:18:10] Quantum plus other devices.
[00:18:10 - 00:18:13] So for the quantum part, if the vendor, obviously,
[00:18:13 - 00:18:18] it's close source information if that level is not allowed
[00:18:18 - 00:18:23] or available, we would just use the underlying available
[00:18:23 - 00:18:27] runtime on the vendor, so vendor-specific runtime,
[00:18:27 - 00:18:31] on being able to communicate between multiple devices,
[00:18:31 - 00:18:36] however we use space RT to support the rest of the.
[00:18:37 - 00:18:43] Yes, so my problem is that I see an incoming clash of level of abstraction
[00:18:43 - 00:18:48] because so all the stuff you're doing here is very low level
[00:18:48 - 00:18:52] while quantum stuff would probably be a lot higher level than this.
[00:18:53 - 00:18:57] So if you have like latency constraints, for example,
[00:18:57 - 00:19:02] but you have to go to a very high abstraction layer to even access the quantum hardware.
[00:19:02 - 00:19:08] I think this would be quite difficult to achieve like a synchronization there.
[00:19:11 - 00:19:18] I mean, I see this working if you had access to like the on chip controllers for quantum device,
[00:19:18 - 00:19:21] but I don't think that's a realistic assumption.
[00:19:22 - 00:19:25] Yeah, I think that's fair, but...
[00:19:27 - 00:19:36] I'm wondering if QDMA and some of the other layers would allow us to
[00:19:36 - 00:19:42] unify at least some level of capabilities around the runtime and you can still
[00:19:42 - 00:19:49] express some of it but in terms of latency constraints there might be...
[00:19:49 - 00:19:54] So there are other latencies that we're not talking about here. There could be
[00:19:54 - 00:19:59] latencies depending on the classical computers as well that are clearly like
[00:19:59 - 00:20:02] anyone's kind of
[00:20:00 - 00:20:04] of playing field that is not well defined yet.
[00:20:04 - 00:20:06] So if you want to do, for example,
[00:20:06 - 00:20:08] quantum error correction on another system
[00:20:08 - 00:20:11] and that system is open source,
[00:20:11 - 00:20:13] what does that latency requirement look like?
[00:20:13 - 00:20:16] How do you launch work on that?
[00:20:16 - 00:20:17] What does the typical exchange look like
[00:20:17 - 00:20:20] between the quantum computer and that device?
[00:20:22 - 00:20:24] I'm interested in what your thoughts are
[00:20:24 - 00:20:26] on that aspect of it,
[00:20:26 - 00:20:29] like where could something like this, but in maybe.
[00:20:30 - 00:20:34] Yeah, I'm not super clear on this myself.
[00:20:34 - 00:20:38] This is just, I'm asking this because I'm trying to understand if you have a clearer
[00:20:38 - 00:20:43] picture than I do because I'm struggling with the same problem that I don't know, what
[00:20:43 - 00:20:45] is this going to be.
[00:20:45 - 00:20:53] And I think for Q2M, I, they're trying to unify some of these things, but then again,
[00:20:53 - 00:20:58] I think there's a different level of abstraction because there's something that lives already
[00:20:58 - 00:21:04] on top of quantum devices, maybe John, you can add something else.
[00:21:06 - 00:21:11] Of course, I mean, this is maybe more speculative as well, but I think maybe the reason why many
[00:21:11 - 00:21:14] vendors don't give you the lower level access is because they want to be able to do things like
[00:21:14 - 00:21:20] calibrations or like load the items in. Things that also cause other latency constraints.
[00:21:22 - 00:21:26] I don't know if those were already priced in, I didn't want to sort of jump ahead of the presentation,
[00:21:26 - 00:21:32] but there might be some concerns that of why it's kind of hard to integrate at this lower level
[00:21:32 - 00:21:42] versus like the higher level that had a QD of i to throw in my offer.
[00:21:56 - 00:22:15] Yeah, that makes sense.
[00:22:15 - 00:22:22] I think it might be, I think one thing that I have been struggling with or have not been
[00:22:22 - 00:22:29] able to communicate as well is who the user is. It might be a good idea to put some sort of
[00:22:29 - 00:22:35] doc together of various different types of users for different distractions because there are levels,
[00:22:36 - 00:22:43] there are quite a lot of levels to this because even in some of the classical like world, you don't
[00:22:43 - 00:22:49] typically, not everyone would typically write the kernels themselves that will call this the runtime
[00:22:49 - 00:22:54] stuff for efficiency and performance and whatnot or latency requirements, right?
[00:22:54 - 00:23:00] They would just call a Kiskit level or some higher level API that just does that thing for you
[00:23:01 - 00:23:10] and builds that workload for you. And often the two get kind of mixed stuff together a lot.
[00:23:19 - 00:23:33] Okay, yes, I mean this is an important point to make because an application sort of developer
[00:23:33 - 00:23:44] wants to access the device to run my quantum algorithms that are built on top of, say,
[00:23:44 - 00:23:55] error-corrected qubits, I do not want to need to know how the error... unless I need to
[00:23:55 - 00:24:08] know. But in general, I do not want the details of the underlying error correction code implementation
[00:24:08 - 00:24:15] exposed. So we have two different abstraction layers.
[00:24:15 - 00:24:24] The one that sees the logical representation of the hardware,
[00:24:24 - 00:24:30] and then the one that actually sees the physical layer.
[00:24:30 - 00:24:44] especially if we go to the era of error-corrected qubits, these two will be separate.
[00:24:44 - 00:24:55] In current hardware, your physical qubits are the real qubits.
[00:24:55 - 00:25:00] Of course I don't care even now I don't care about how do you...
[00:25:00 - 00:25:09] implement the pulses or whatever you need to do for the specific physical implementation.
[00:25:09 - 00:25:20] But it still is a different interacting with the actual physical qubit in the sense.
[00:25:20 - 00:25:29] In the error corrected era, I want to interact with the logic for qubit, not with actual
[00:25:29 - 00:25:40] or dozens or hundreds of physical qubits that implement this error correct at logical
[00:25:40 - 00:25:41] qubit.
[00:25:41 - 00:25:52] This one makes sense.
[00:25:52 - 00:26:01] And there's definitely also different latency requirements between these two, what you want
[00:26:01 - 00:26:06] to do for the, again, for dealing with error correction codes themselves.
[00:26:06 - 00:26:21] You have very stringent latency and real-time requirements for the actual compute that's
[00:26:21 - 00:26:32] I mean, still a concern, but less, less detrimental if you do not meet it exactly.
[00:26:51 - 00:27:08] So that relates to how the actual kernel, like the analogous term here is the kernel,
[00:27:08 - 00:27:10] how the kernel is written.
[00:27:10 - 00:27:17] So that combination phase is completely separate from how the work gets to the device itself
[00:27:17 - 00:27:19] or launched to the device.
[00:27:19 - 00:27:25] It is related, but the compiler is doing that all on its own.
[00:27:25 - 00:27:31] That's compiler's job to make those logical and physical connections and how the mapping
[00:27:31 - 00:27:37] is going to work or within the kernel itself.
[00:27:37 - 00:27:44] The execution of the kernel itself is the boundary at which the runtime differentiates
[00:27:44 - 00:27:51] from the compiler's task.
[00:27:51 - 00:28:00] The compiler would most definitely come first in terms of compiling the said implementation
[00:28:00 - 00:28:07] or whatever that's going to get executed on the QPU and the runtime will be responsible
[00:28:07 - 00:28:14] for then taking that thing and putting it on the perspective system.
[00:28:14 - 00:28:15] Yeah.
[00:28:28 - 00:28:33] So that is, from my perspective, that sounds more like a programming model.
[00:28:33 - 00:28:41] like how do I actually tell the compiler express these qubits? That to me is some sort of a language
[00:28:41 - 00:28:51] job, like some language expressing what those like mappings look like or not the physical part,
[00:28:51 - 00:28:58] just how to express the logical qubits and what am I doing with them in the first place.
[00:29:03 - 00:29:29] Yeah.
[00:29:33 - 00:29:56] Yes.
[00:29:56 - 00:30:00] Yeah, yeah, yeah, sorry.
[00:30:00 - 00:30:01] It doesn't have to be though.
[00:30:01 - 00:30:05] It's just how we write some of those programs today.
[00:30:05 - 00:30:11] But you could most definitely express the shader code, which is basically compiled into
[00:30:11 - 00:30:17] some intermediate language, like P-tax or something, like assembly code, that can be
[00:30:17 - 00:30:24] completely separately compiled and shipped and maintained from the launch part itself.
[00:30:24 - 00:30:29] So the launch part is typically the driver or what we call the launch driver or whatever,
[00:30:29 - 00:30:33] can be written separately, also compiled separately,
[00:30:33 - 00:30:37] to wherever you want to map that to.
[00:30:39 - 00:30:40] Does that kind of make sense?
[00:30:44 - 00:30:45] Yeah, me.
[00:30:54 - 00:30:55] Yeah.
[00:30:59 - 00:31:05] Yes, that's when it gets launched.
[00:31:05 - 00:31:06] Yes, yes, yes.
[00:31:06 - 00:31:13] By the application.
[00:31:13 - 00:31:15] Yes, precisely.
[00:31:15 - 00:31:16] Yes.
[00:31:16 - 00:31:17] Yes.
[00:31:17 - 00:31:45] That is the way it works.
[00:31:45 - 00:31:50] that is defined by the person building the application or the algorithm, right?
[00:31:50 - 00:31:54] Like that's the only person that knows like how these things are going to actually
[00:31:54 - 00:32:00] stitch together unless there's like always a clear dependency that you can just resolve,
[00:32:00 - 00:32:07] which can be done in a compiler. But if there's not like always a guaranteed dependency,
[00:32:07 - 00:32:13] then you're asking like one scientist to be to say like, hey, I know that I want to
[00:32:13 - 00:32:16] build this workload and this workload is going to launch a bunch of stuff and
[00:32:16 - 00:32:19] that stuff is going to be launched on quantum computers it might be launched
[00:32:19 - 00:32:26] in heterogeneous like platforms where there's CPU, GPU, whatever and I will call
[00:32:26 - 00:32:31] these like kernels or COMB, COMB, COMB, C, whatever and build that
[00:32:31 - 00:32:35] bag myself because only I know the correct order in which they need to
[00:32:35 - 00:32:40] happen to build this workload or this application and once that application
[00:32:40 - 00:32:46] is built, it can be further modular that gets shipped within, for example, something like
[00:32:46 - 00:32:51] KissKit. And someone on top of KissKit can use it to build larger applications, larger
[00:32:51 - 00:32:59] workloads, and would have no idea how the internal launching of various different DAG
[00:32:59 - 00:33:04] thing works. They just know that I need to call this one thing.
[00:33:04 - 00:33:11] Correct, yeah.
[00:33:11 - 00:33:18] Yes.
[00:33:18 - 00:33:23] Yes.
[00:33:23 - 00:33:34] Yes. Yes. Yeah. Yeah.
[00:33:34 - 00:33:37] Often, performance code does exactly that.
[00:33:37 - 00:33:39] So if you want to write like really,
[00:33:39 - 00:33:41] really hyper-performance code,
[00:33:41 - 00:33:44] you would typically the person working,
[00:33:44 - 00:33:47] the writing the high-level application kind of
[00:33:47 - 00:33:50] owns the entire space and squishes them together.
[00:33:50 - 00:33:53] And why you might want to squish them together is
[00:33:53 - 00:33:56] because in the previous case, you didn't know anything
[00:33:56 - 00:33:59] about the DAG that was within each one of those tiny things.
[00:33:59 - 00:34:02] There might be overlap between the tiny DAG inside that
[00:34:02 - 00:34:06] and some other tiny DAGs that are in the larger application.
[00:34:06 - 00:34:08] And you might wanna fuse them together
[00:34:08 - 00:34:10] or overlap them better to write
[00:34:10 - 00:34:12] like really high-performance code.
[00:34:13 - 00:34:17] But that's when you wanna go from like 80 to like 99%
[00:34:17 - 00:34:20] of the performance of that high gap
[00:34:20 - 00:34:22] that you might want to close.
[00:34:22 - 00:34:27] And in some cases, it never makes sense to like squish them together because the operation
[00:34:27 - 00:34:37] always is sequential or some level of dependencies always like present that you know how to do.
[00:34:37 - 00:34:42] Sorry, it's like a very complex space that I'm trying to define, but the runtime itself
[00:34:42 - 00:34:44] is in that convey a copy.
[00:34:44 - 00:34:50] Like that's the part that I personally am talking about right now.
[00:34:50 - 00:34:52] up.
[00:35:00 - 00:35:19] Yeah, precisely.
[00:35:19 - 00:35:26] So I think without looking at the full end-to-end flow, there is no complete story, right?
[00:35:26 - 00:35:33] We have to do that, but that includes the compiler works stream because they would be
[00:35:33 - 00:35:38] the ones that are actually compiling the code that will get executed in some capacity.
[00:35:38 - 00:35:46] That includes whatever higher level language is allowed to express some of these comp
[00:35:46 - 00:35:48] a comp b algorithms.
[00:35:48 - 00:35:53] And it includes what gets built on top of it, which may be something like his kit or his
[00:35:53 - 00:35:59] get that stitches them together in some regards to create the end-to-end flow, which is where
[00:35:59 - 00:36:08] most of maybe performance timing or some sort of result analysis happens.
[00:36:08 - 00:36:16] It's kind of like my view of the whole space.
[00:36:16 - 00:36:29] No, no, this is, this is, yeah, this is very useful and what some of the other people
[00:36:29 - 00:36:36] have pointed out on the, on the closed nature of some of the runtime associated with quantum
[00:36:36 - 00:36:42] is also a very important question because there might be a, an argument for some sort
[00:36:42 - 00:36:45] So I don't want to call it a standard,
[00:36:45 - 00:36:51] but some level of a standard, which exposes low level
[00:36:51 - 00:36:54] mechanisms that still allow you to build a runtime,
[00:36:54 - 00:36:56] like this on top of it.
[00:36:56 - 00:36:59] But that requires, obviously, vendor partnership
[00:36:59 - 00:37:02] and a lot more than what I'm talking about.
[00:37:02 - 00:37:04] But I can envision in a perfect world that
[00:37:04 - 00:37:09] could exist in some capacity.
[00:37:09 - 00:37:11] like there is an abstraction that sits on top of that.
[00:37:29 - 00:37:30] Yeah.
[00:37:31 - 00:37:32] Yeah.
[00:37:32 - 00:37:34] Let me just, if I can,
[00:37:34 - 00:37:37] give me one second, I'm gonna add a slide
[00:37:37 - 00:37:44] And then, actually, I have a perfect thing for this.
[00:37:53 - 00:37:57] Sorry, it gave me just one second as I published.
[00:37:57 - 00:38:03] There is this article that just recently got published.
[00:38:03 - 00:38:07] I actually really, really liked this.
[00:38:07 - 00:38:12] It is a AI chip architecture comparison.
[00:38:12 - 00:38:17] So it talks about essentially a ton of different architectures
[00:38:17 - 00:38:20] that have come out around AI space.
[00:38:20 - 00:38:22] And what are some of the commonalities
[00:38:22 - 00:38:24] and what are their various different bets
[00:38:24 - 00:38:29] on differentiations to essentially do better
[00:38:29 - 00:38:31] in this AI space?
[00:38:31 - 00:38:34] The point in this meeting isn't necessary
[00:38:34 - 00:38:36] to talk about the underlying architecture.
[00:38:36 - 00:38:41] to talk about how some of the similarities allow you to still program them as if they
[00:38:41 - 00:38:46] were similar looking things, but they are very, sometimes they're very different.
[00:38:46 - 00:38:57] So there's like stuff like NVIDIA GPUs, which I'll show the pictures of two things.
[00:38:57 - 00:39:00] So these are very similar looking architectures.
[00:39:00 - 00:39:09] So what GPUs say is that we will create a distinction about what is sort of the memory
[00:39:09 - 00:39:16] hierarchy and what is a compute hierarchy and kind of give you a sense of both and control
[00:39:16 - 00:39:23] for both that you can map logical entities to.
[00:39:23 - 00:39:28] So this is what NVIDIA's thing looks like and this is what AMD's thing looks like,
[00:39:28 - 00:39:33] and you can see they're very like even just with a block diagram quickly looking at it
[00:39:33 - 00:39:34] with no contact.
[00:39:34 - 00:39:38] They look very similar.
[00:39:38 - 00:39:42] The reason why is because the problems that they're solving is obviously very similar
[00:39:42 - 00:39:47] so they end up being similar looking architectures but it's a bunch of like these HBM's which
[00:39:47 - 00:39:54] are the memory, the giant like memory spaces at the very slowest top end of it with a bunch
[00:39:54 - 00:39:57] to couple together with a bunch of faster memory,
[00:39:57 - 00:40:00] like the L2 cache, and then there is like
[00:40:00 - 00:40:00] the
[00:40:00 - 00:40:06] darker green boxes which represent like a local, some sort of local cash within
[00:40:06 - 00:40:12] the GPU. And the hierarchy looks very similar even though it is slightly
[00:40:12 - 00:40:16] different but very similar looking hierarchy because they're again solving
[00:40:16 - 00:40:21] the same issue problem. There are other chips that are not similar looking.
[00:40:21 - 00:40:28] They are more spatial chips like this thing.
[00:40:28 - 00:40:33] Maybe there's a better one that I can show.
[00:40:33 - 00:40:35] This article is amazing by the way.
[00:40:35 - 00:40:41] Like maybe something like this which talks about a more like,
[00:40:41 - 00:40:43] this is a seabird ship.
[00:40:43 - 00:40:47] So it talks about more of how if the compute and the data fabric
[00:40:47 - 00:40:52] was like all together and there wasn't like this L2 caches or caches that you
[00:40:52 - 00:40:58] relied on and things had to perfectly flow from one corner to another to do a
[00:40:58 - 00:41:04] task really really well. Regardless of how the underlying architecture is
[00:41:04 - 00:41:11] represented, you can always think about all of these chips as logical work items
[00:41:11 - 00:41:17] that's that gets scheduled on top of it and some sort of data movement. So there
[00:41:17 - 00:41:23] two core principles, how work gets mapped, which is the compute part of it, and how data
[00:41:23 - 00:41:29] movement works, which is the memory part of it, and the two combined together to make
[00:41:29 - 00:41:37] a software ecosystem which includes the programming language, the compilers, and the runtime,
[00:41:37 - 00:41:39] all three large components.
[00:41:39 - 00:41:43] And then you, then that gets mapped to the typical frameworks like Torch, and you use
[00:41:43 - 00:41:49] towards to create cool applications and do AI stuff on top of it. I know that's
[00:41:49 - 00:41:55] inherently very different from how quantum computers work. However, if you
[00:41:55 - 00:41:59] think of quantum computers in scope of the entire application, my thought
[00:41:59 - 00:42:04] process is they fit in a very similar world where they need to do their like
[00:42:04 - 00:42:10] the specialized accelerator. Something similar like the W and M.A. course,
[00:42:10 - 00:42:15] There's not a good picture for this, but imagine this thing on the right hand side
[00:42:16 - 00:42:21] Inside one of the AMD chips or NVIDIA chips. There's saying there's this thing called matrix score or tensor cores
[00:42:22 - 00:42:23] also
[00:42:23 - 00:42:28] accessible to WMMA instructions just one simple instruction that says
[00:42:28 - 00:42:33] Launch an insane amount of matrix multiplication on top of these computers
[00:42:33 - 00:42:37] And I don't care how that happens internally the hardware takes care of it
[00:42:37 - 00:42:43] The Q-Bits are handled by the quantum computer like themselves in some capacity.
[00:42:43 - 00:42:47] All I know is how to schedule an operation on top of it.
[00:42:47 - 00:42:54] And maybe that's a good programming language, programming model to kind of talk about quantum computers with the TBD.
[00:42:54 - 00:42:59] But the idea is that it's an accelerator, much like WMMA course,
[00:42:59 - 00:43:04] which most programmers don't understand how it properly works.
[00:43:04 - 00:43:11] but all I know is within my kernel, I could do a lot of general purpose compute, like load things
[00:43:11 - 00:43:18] from this low memory to fast memory, do some reductions on top of it, use my scalar cores,
[00:43:18 - 00:43:24] use my vector cores, and when it's time to do this really, this one operation that it does really,
[00:43:24 - 00:43:29] really, really, really well called the matrix cores, I will handle it, call the, call the gem,
[00:43:29 - 00:43:32] I will hand it off to the matrix scores.
[00:43:32 - 00:43:36] And then all I know is how to put things in,
[00:43:36 - 00:43:39] which is the inputs, and how to what to expect as an output.
[00:43:39 - 00:43:41] And I will take that output and couple it together
[00:43:41 - 00:43:44] with other things to make my entire application,
[00:43:44 - 00:43:47] make my entire kernel, like general matrix multiplication
[00:43:47 - 00:43:50] kernel or attention kernel, and take that kernel which
[00:43:50 - 00:43:56] comp A and comp B and put that into an AI workload that
[00:43:56 - 00:43:57] knows how to stitch them together
[00:43:57 - 00:44:02] and do basically build an LLM out of that.
[00:44:02 - 00:44:05] That's the hard flow of how things work on this base.
[00:44:05 - 00:44:10] And by the way, once you have built that LLM model,
[00:44:11 - 00:44:15] that is mostly most of the main work happens on the GPU,
[00:44:15 - 00:44:19] you still have CPU threads orchestrating
[00:44:19 - 00:44:22] when like input streams from your keyboard
[00:44:22 - 00:44:25] goes into a cursor window or a cloud window,
[00:44:25 - 00:44:29] and what that text turns into after pre-processing
[00:44:29 - 00:44:32] into some sort of tokens, and how those tokens feed
[00:44:32 - 00:44:36] into the actual kernel, and how to read the algorithm
[00:44:36 - 00:44:37] of the tokens.
[00:44:37 - 00:44:40] So much of that part happens on the CPU side.
[00:44:40 - 00:44:45] The tool calls the text to voice or voice to text
[00:44:45 - 00:44:48] or many of the other things that we use,
[00:44:48 - 00:44:54] and the LLM parts happens inside the GPU itself.
[00:44:54 - 00:44:57] Inside the GPU, the matrix part happens on the matrix form.
[00:44:57 - 00:45:00] So it's like a very, I don't know, special.
[00:45:00 - 00:45:07] task for specialized things. Very heterogeneous as much as we try not to
[00:45:07 - 00:45:15] make it look like heterogeneous. I don't know if that clarified things or
[00:45:15 - 00:45:17] I haven't made it worse, but...
[00:45:26 - 00:45:28] Oh yeah, this is awesome, yeah.
[00:45:45 - 00:45:58] Yeah, I think that that is an important question though.
[00:45:58 - 00:46:09] So from my perspective right now, what's happening is QPU is like a specialized accelerator in
[00:46:09 - 00:46:12] in some sense right now.
[00:46:12 - 00:46:17] The vendor owns the runtime and a good bit of the stack.
[00:46:17 - 00:46:20] And most of it is not visible, right?
[00:46:20 - 00:46:22] Like most of it is just handled.
[00:46:22 - 00:46:26] And some of that is because of the underlying inherent
[00:46:26 - 00:46:28] requirements of the hardware itself.
[00:46:28 - 00:46:30] Like there are certain latency requirements.
[00:46:30 - 00:46:34] There are certain things checkboxes need to be checked.
[00:46:34 - 00:46:39] From the user's perspective, you send things in
[00:46:39 - 00:46:41] and you get an output out.
[00:46:41 - 00:46:44] And that output, we might know how to process
[00:46:44 - 00:46:45] or understand that output.
[00:46:45 - 00:46:48] And we might know how to what to send in.
[00:46:48 - 00:46:52] That's the perspective we have or I have
[00:46:52 - 00:46:56] of how we are exposing some of the staff.
[00:46:56 - 00:46:59] And there might be some tools built on top of it,
[00:46:59 - 00:47:02] something like his gate or vendor specific tools
[00:47:02 - 00:47:06] that allow you to have different types of operations here,
[00:47:06 - 00:47:09] like WMMA, which is the matrix course,
[00:47:09 - 00:47:11] but like something akin to that,
[00:47:11 - 00:47:13] but maybe a handful of them,
[00:47:13 - 00:47:16] or maybe like a good bit of them, right?
[00:47:16 - 00:47:19] And that's all vendor provided.
[00:47:19 - 00:47:22] Here are our recipes for things you might want to use
[00:47:23 - 00:47:27] the qubits for, or like how you might want
[00:47:27 - 00:47:29] to manipulate certain things.
[00:47:29 - 00:47:31] And it's your job to orchestrate them
[00:47:31 - 00:47:35] in certain order to construct something meaningful.
[00:47:35 - 00:47:40] And then on this sideline lives like a GPU, a CPU,
[00:47:40 - 00:47:44] an FPGA might be closer, and then some links that transfer
[00:47:44 - 00:47:46] things back and forth, because you might still
[00:47:46 - 00:47:49] want to build a large application that
[00:47:49 - 00:47:50] has all of these components.
[00:48:01 - 00:48:02] Yeah.
[00:48:15 - 00:48:19] Yeah, that's just, either or is fine.
[00:48:19 - 00:48:24] Like some abstraction that hides the underlying complexity of,
[00:48:25 - 00:48:26] yes, yeah.
[00:48:26 - 00:48:33] Yes.
[00:48:33 - 00:48:40] I mean, I really see levels here.
[00:48:40 - 00:48:48] So there is the physical control, the pulses and so, and that is probably out of scope for
[00:48:48 - 00:48:52] what we are talking about, but I'm not sure.
[00:48:52 - 00:49:03] But that is something really very close to the hardware with very precise timing requirements.
[00:49:03 - 00:49:08] And accuracy really depends on having exact anchor timings.
[00:49:08 - 00:49:19] Then there is probably this sort of level that is still very close to the QPUs, but
[00:49:19 - 00:49:26] builds on top of that in a sense, where you would implement surface codes and things like that.
[00:49:27 - 00:49:36] So error correcting codes. And that is also what probably live there on what way you've drawn this
[00:49:36 - 00:49:46] FPGA thing. So where you really need very low latencies and have real-time requirements.
[00:49:46 - 00:50:00] And then you have your application codes that sort of uses these effective blocks of hard...
[00:50:00 - 00:50:07] where and their control and that is what sits probably on the left side of your picture there.
[00:50:07 - 00:50:14] Yeah, yeah, I agree. I think there are levels. Also, I would stress that so the real-time
[00:50:14 - 00:50:21] requirements that you mentioned, Marcus, something similar also exists on the GPU side, but not
[00:50:21 - 00:50:29] from the physics point of view, but how operations, when operations are expected to arrive. A compiler
[00:50:29 - 00:50:34] handles a good benefit, but handles it often in a very inefficient way. And what I mean by that is
[00:50:34 - 00:50:42] in a GPU, when you issue a memory instruction to the, like maybe a load from the, from like a
[00:50:42 - 00:50:50] HBM of a data that I want to process, there needs to be a memory model that explains when this load
[00:50:50 - 00:50:57] is expected. And that allows me to make sure that when I touch the registers that now have this data
[00:50:57 - 00:51:04] actually have the right set of data because if there isn't a like some weight counts or some
[00:51:04 - 00:51:10] memory model or some way to ensure that my data has arrived some synchronization mechanism,
[00:51:11 - 00:51:17] then I could just I will be getting garbage out from the other side because I could be reading
[00:51:17 - 00:51:22] garbage in one run, I could be reading real data in another run like I have no idea. So we have
[00:51:22 - 00:51:27] notions of memory models to ensure that that happens and that's not just GPUs like
[00:51:27 - 00:51:34] CPU's x86 has its own requirements of similar sort of thing.
[00:51:34 - 00:51:40] I mean, again, even if we write a where to write assembly
[00:51:40 - 00:51:47] on an x86, there's all this sort of internal interpretation
[00:51:47 - 00:51:53] compilation to micro-instruction that happens,
[00:51:53 - 00:51:58] that deal with all these timing and hardware.
[00:52:02 - 00:52:03] That's precisely.
[00:52:03 - 00:52:04] Yes.
[00:52:04 - 00:52:09] And what I'm stressing is that they're not very different.
[00:52:09 - 00:52:12] Like if you look at it from that view,
[00:52:12 - 00:52:13] we still have those timing requirements
[00:52:13 - 00:52:16] of when certain instructions are going to issue,
[00:52:16 - 00:52:18] when certain data is going to arrive.
[00:52:18 - 00:52:21] So there could be a view,
[00:52:21 - 00:52:25] But most people never care about it.
[00:52:25 - 00:52:28] When they write a C++ code, they're not thinking of,
[00:52:28 - 00:52:32] hey, I need to worry about the timing in which
[00:52:32 - 00:52:34] my instructions are going to be scheduled,
[00:52:34 - 00:52:35] or this data is not going to be there,
[00:52:35 - 00:52:39] or some sort of that really, really, really low-level
[00:52:39 - 00:52:41] timing requirements.
[00:52:41 - 00:52:43] The assumption is the hardware vendor
[00:52:43 - 00:52:49] has taken care of that for you, bar, like, buns, and whatnot.
[00:52:49 - 00:52:53] But most of that is taking care of that for you.
[00:52:53 - 00:52:56] And the same assumption is true for QPUs as well.
[00:52:56 - 00:52:59] Much of that is taken care of by the vendor.
[00:52:59 - 00:53:02] And it's a question of what is an abstraction
[00:53:02 - 00:53:06] that they can expose, which then therefore,
[00:53:06 - 00:53:10] the compiler can target, or the runtime can target,
[00:53:10 - 00:53:12] and everything else can target.
[00:53:12 - 00:53:15] And that's kind of where I would like to sit with the stuff
[00:53:15 - 00:53:16] I'm doing.
[00:53:19 - 00:53:37] So, in the CPU GPU world, in the classical world, I have control over exactly where I'm
[00:53:37 - 00:53:42] going to place my computation, when I'm going to place it, how I'm going to place it, and
[00:53:42 - 00:53:44] to which device I'm going to place it.
[00:53:44 - 00:53:47] Like I have all of that control.
[00:53:47 - 00:53:51] The same is not true for TPU right now.
[00:53:51 - 00:53:52] On the key.
[00:54:17 - 00:54:31] Yeah, sorry, maybe, yeah, maybe I should say it's not at the same granularity.
[00:54:31 - 00:54:34] It is the same in terms of targeting the device itself.
[00:54:34 - 00:54:40] Like, yes, I am deciding I'm going to launch this work onto the QPU using vendor specific,
[00:54:40 - 00:54:46] vendor specific APIs, but it's not at a fine grain granularity.
[00:54:46 - 00:54:49] And maybe that's because the hardware itself
[00:54:49 - 00:54:52] doesn't allow you to be at a fine grain granularity.
[00:54:52 - 00:54:56] And a great example of that, it would be if I take the WMMA
[00:54:56 - 00:55:00] instruction, it expects the registers to be laid out
[00:55:00 - 00:55:01] You know.
[00:55:00 - 00:55:06] specific way. I don't have a control over, hey, today I'm going to lay it out differently.
[00:55:06 - 00:55:11] That's just not a thing because the WMMA instruction is a spatial kind of architecture
[00:55:11 - 00:55:16] that expects the inputs in a certain way and expects the output in a certain way. That's
[00:55:16 - 00:55:24] just the how the hardware is designed. I can't say, I'm going to instead of placing my A
[00:55:24 - 00:55:28] in a nice contiguous manner, I'm going to place half of A here and half of A here and
[00:55:28 - 00:55:30] expect correct answers on the other end.
[00:55:30 - 00:55:34] And even though I have control to do that because it's just
[00:55:34 - 00:55:38] registers that I'm writing to, I might not even have that
[00:55:38 - 00:55:41] control on the key view, so I'd let alone get the correct
[00:55:41 - 00:55:42] result out.
[00:55:49 - 00:55:49] Yeah.
[00:55:58 - 00:55:59] Okay, cool.
[00:56:08 - 00:56:09] Yeah.
[00:56:09 - 00:56:19] Could, could, I think it would be really useful for, for me to understand what that topology looks like and how, what APIs allow that to be exposed.
[00:56:19 - 00:56:26] Or just as an example, we could pick any one specific implementation.
[00:57:19 - 00:57:42] I'm best specific.
[00:58:12 - 00:58:42] That's really cool.
[00:58:42 - 00:58:46] This is exactly what is missing from my...
[00:58:46 - 00:58:48] ...twides.
[00:58:48 - 00:58:53] I'd love to think about a way to...
[00:58:53 - 00:58:55] Even if it's...
[00:58:55 - 00:59:01] To me, what Marcus just mentioned on a specific day, it sounds like some level of dynamic
[00:59:01 - 00:59:09] behavior in the topology itself or in the representation itself, which is okay.
[00:59:09 - 00:59:14] You could still build a very nice logical abstraction on top of it that you targeted,
[00:59:14 - 00:59:15] right?
[00:59:15 - 00:59:18] But that's exactly what I'm describing with MMA's.
[00:59:39 - 01:00:00] So, right now are they targeting the physical...
[01:00:00 - 01:00:01] Cubits.
[01:00:30 - 01:00:49] Yeah, to me it sounds like at least right now because of the nature of how this structure
[01:00:49 - 01:00:54] is structured, they're doing both the runtime component of it.
[01:00:54 - 01:00:58] Or maybe they're doing the compilation and like how does the work then gets launched
[01:00:58 - 01:01:05] onto these, they're probably doing that part as well. There's less of a separation there.
[01:01:05 - 01:01:09] That's what it sounds like to me.
[01:01:09 - 01:01:16] Yeah.
[01:01:16 - 01:01:27] Yeah, so we have a component. Yeah, so we have a component called target that lets you specify
[01:01:27 - 01:01:34] specify which underlying compiler you want to use for a specific implementation or
[01:01:34 - 01:01:38] like a specific piece of it. So maybe think of like one of the config copies.
[01:01:38 - 01:01:43] And then once that's compiled you then launch it on that specific target that
[01:01:43 - 01:01:50] you compiled for using a specific compiler. So it sits underneath the
[01:01:50 - 01:01:55] the combinations well not not not necessarily saying that is
[01:01:57 - 01:02:00] They could be that is like the only solution, but that is
[01:02:01 - 01:02:03] What we are looking at
[01:02:04 - 01:02:06] Sorry, there's a question back
[01:02:08 - 01:02:14] I think there's just coming back to what I said earlier is just exactly the point I was phrasing
[01:02:14 - 01:02:23] where this difference is I think, because again, I think once we have compiled and launched
[01:02:23 - 01:02:29] in your terms the circuit or the instructions we're going to run, this usually ends up in
[01:02:29 - 01:02:35] the closed part of the quantum vendor at the moment, because for most of them this is basically
[01:02:35 - 01:02:41] the secret ingredient of how they will implement their gates, sample even to what you do a generic
[01:02:41 - 01:02:47] type of compilation and then after this is submitted they will even do a second
[01:02:47 - 01:02:51] phase where they compile to something that matches their hardware instructions
[01:02:51 - 01:02:58] and I'm not really sure if we can in check in between compiling and then
[01:02:58 - 01:03:08] running stuff. That makes sense I would like to so I don't have an answer for
[01:03:08 - 01:03:17] this right now or I'd love to explore this what this looks like and if there is a way
[01:03:17 - 01:03:23] to inject something in between that calls like a proprietary thing in the back end I'm
[01:03:23 - 01:03:27] not sure what that looks like John Eel said a question.
[01:03:27 - 01:03:31] I do know it's hard for me to comment on too much because it's not really my own activities
[01:03:31 - 01:03:35] and like what we're doing and continuing with this stuff but obviously for our runtime we
[01:03:35 - 01:03:38] We do need to call to classical hardware, and I'm not entirely sure how they do it.
[01:03:38 - 01:03:43] I wish I had brought someone along who had a better opinion.
[01:03:43 - 01:03:48] My understanding, I guess, is that we compiled down to something like LLBN, and there are
[01:03:48 - 01:03:53] tools to do the synchronization between the quantum kernel and some classical computation
[01:03:53 - 01:03:54] attending at the same time.
[01:03:54 - 01:04:00] But that seems like it maps in a bit closer to what you're talking about with having some
[01:04:00 - 01:04:03] classical run time to then walk into the quantum kernel.
[01:04:05 - 01:04:07] So I can kind of see how it would work,
[01:04:07 - 01:04:08] but yeah, it's not something that loads
[01:04:08 - 01:04:10] that does do as far as I'm all that.
[01:04:10 - 01:04:13] Yeah, yeah, what I'm also thinking is like
[01:04:13 - 01:04:16] if you are going to construct more complex systems
[01:04:16 - 01:04:19] where you have other devices there,
[01:04:20 - 01:04:22] with the closer, the closer it's going,
[01:04:22 - 01:04:24] one can still very much exist,
[01:04:24 - 01:04:27] that is the secret recipe,
[01:04:27 - 01:04:29] but there still needs to be a contract
[01:04:29 - 01:04:31] between that and everything else in the system
[01:04:31 - 01:04:34] that the company or vendor doesn't own.
[01:04:34 - 01:04:36] What does that contract look like?
[01:04:36 - 01:04:38] It's not very clear to me.
[01:04:38 - 01:04:39] Yeah, so I guess in our case,
[01:04:39 - 01:04:41] we only have like the hugger representation
[01:04:41 - 01:04:42] of the open source.
[01:04:42 - 01:04:43] Presumably you could have a compiler
[01:04:43 - 01:04:45] that took the hugger representation,
[01:04:45 - 01:04:48] compiled it to something LOBM-like
[01:04:48 - 01:04:50] that had the tools to the space RT
[01:04:50 - 01:04:52] instead of using our runtime, for example.
[01:04:53 - 01:04:57] So you would need some compiler that compiled
[01:04:57 - 01:04:58] the high level representation of something
[01:04:58 - 01:05:00] that then made this lot of the tools to be about.
[01:05:00 - 01:05:05] what about that? Is that the right understanding? Yeah, that is the right understanding of what
[01:05:05 - 01:05:12] I have been thinking about at least. But it's a fair point that there might be certain vendor
[01:05:12 - 01:05:19] specific close source things that do make it really hard to do some of that and you might just
[01:05:19 - 01:05:25] be living at a point that is just high level API has been called for the quantum piece and maybe
[01:05:25 - 01:05:31] everything else can use something like a heterogeneous runtime. I'm still that's
[01:05:31 - 01:05:35] that part I'm not sure about and maybe that's something I have to like do a
[01:05:35 - 01:05:38] bit more research in. Yeah I mean I guess I wonder if you'd have an easier
[01:05:38 - 01:05:42] server if you were putting this on the control system to keep you to start with.
[01:05:42 - 01:05:46] You're quite ambitious at the moment with like wanting to integrate the
[01:05:46 - 01:05:51] HPE resources and then I'll set like the FPGAs and GPUs as well. Obviously we
[01:05:51 - 01:05:55] you have a machine with GPUs attached and they keep you attached.
[01:05:55 - 01:05:56] And we have a runtime that's on it.
[01:05:56 - 01:05:58] You could do the same thing here.
[01:05:58 - 01:05:58] Yeah.
[01:06:51 - 01:07:06] Yeah, John, do you have a comment?
[01:07:06 - 01:07:11] Yeah, I guess that's just to me that sounds like a slightly different use case.
[01:07:11 - 01:07:13] I think they're both useful.
[01:07:13 - 01:07:17] And like what I'm describing there is kind of something to what we're working on and why
[01:07:17 - 01:07:20] Yeah, and they both fit together.
[01:07:20 - 01:07:21] They're kind of part of it.
[01:07:21 - 01:07:23] Maybe you could imagine a world where all these things have
[01:07:23 - 01:07:27] like some unified program or even going to do stuff
[01:07:27 - 01:07:28] close to the device where necessary.
[01:07:28 - 01:07:32] But yeah, I think both of those things
[01:07:32 - 01:07:34] are useful use cases to some degree.
[01:07:36 - 01:07:39] Yeah, I kind of, sorry, good.
[01:07:40 - 01:07:42] Oh, I kind of agree with that as well.
[01:07:42 - 01:07:47] So for me, it's different use cases,
[01:07:47 - 01:07:55] but also that is an easier problem to solve because you do
[01:07:55 - 01:08:01] essentially just say, hey, let the QPU stack do everything
[01:08:01 - 01:08:02] on its own.
[01:08:02 - 01:08:05] All I'm going to do is potentially just submit stuff to it.
[01:08:05 - 01:08:08] That's the only part that you're doing at that point.
[01:08:08 - 01:08:11] And that's not necessarily a device I'd run time.
[01:08:11 - 01:08:16] that's just our runtime that couples things together
[01:08:16 - 01:08:19] and might have more control over parts of it
[01:08:19 - 01:08:22] and might have less control over the some parts of it.
[01:08:23 - 01:08:26] To me, that's not really a definition
[01:08:26 - 01:08:28] of what our device-side runtime looks like
[01:08:28 - 01:08:30] and it wouldn't account for very different
[01:08:30 - 01:08:35] topological level of optimizations or spatial optimizations
[01:08:35 - 01:08:38] or being able to connect certain pieces together
[01:08:38 - 01:08:42] and optimize for the transfers between the two.
[01:08:44 - 01:08:46] But yes, I agree that is a use case still.
[01:08:49 - 01:08:54] Or you must also require something like that too at the top.
[01:08:57 - 01:08:59] Those are kind of my thoughts on that.
[01:09:08 - 01:09:16] Yeah, and it might be useful, at least in this working group, to talk about what do we care
[01:09:16 - 01:09:18] more about defining?
[01:09:18 - 01:09:26] Or if, I mean, we could do both, but some sort of scope that says this is more feasible
[01:09:26 - 01:09:28] or this is more interesting.
[01:09:28 - 01:09:33] And there are other is entirely like a vendor-specific story.
[01:09:33 - 01:09:40] That is also an argument to be had for the working group.
[01:10:00 - 01:10:24] Yeah, I think what we also need to consider is what the other working groups are doing
[01:10:24 - 01:10:27] in these areas.
[01:10:27 - 01:10:31] like the the resource management working group, I think, or like this
[01:10:31 - 01:10:38] QLIM IQIS stuff. I think we're very much, I should consider their work as
[01:10:38 - 01:10:42] well, because I think they're trying to come up with like a unified scheme to
[01:10:42 - 01:10:49] access hardware. But if they don't consider the model there, we control a
[01:10:49 - 01:10:55] runtime that is sits below the spawnry, this might get complicated to achieve, I
[01:10:55 - 01:11:03] I think it should keep that in mind at least when we try to do this model of the stuff.
[01:11:03 - 01:11:07] I think for the higher level stuff it's a little bit easier because I think the boundaries
[01:11:07 - 01:11:12] are clearer there where stuff ends, at least in my mind, but I think for the longer stuff
[01:11:12 - 01:11:19] should definitely keep other working clips in loop.
[01:11:42 - 01:11:58] Yes, it will be using the... yes, yes. So it will sit on top of that because the runtime
[01:11:58 - 01:12:05] requires all of that information and it doesn't expect to build that information itself.
[01:12:05 - 01:12:11] Typically what happens is the same sort of levels exist within the GPU runtime as well,
[01:12:11 - 01:12:19] things get built on top of like the KFD driver or something like that that says
[01:12:19 - 01:12:25] there's like KFD driver, rock car, and then HSA like all of these various
[01:12:25 - 01:12:31] things report information about what the resources underlying resources look
[01:12:31 - 01:12:37] like which the runtime says okay now I know what I have how do I schedule work
[01:12:37 - 01:12:43] on top of it or how do a launch work on top of it with different level capabilities,
[01:12:43 - 01:12:50] granted the hardware capabilities to allow you to do whatever.
[01:12:50 - 01:12:53] And that same is true for CUDA runtime.
[01:12:53 - 01:12:58] CUDA runtime sits on top of a layer that the driver reports the resources or some level
[01:12:58 - 01:13:05] of thing reports, the resources of whatever is there, the topology, how it's connected,
[01:13:05 - 01:13:05] and so on.
[01:13:16 - 01:13:18] Yeah, I think that's a good question.
[01:13:18 - 01:13:23] I was hoping I think the other working groups
[01:13:23 - 01:13:27] were mentioned, it might be useful to have them share
[01:13:27 - 01:13:30] what they're thinking about exposing potentially
[01:13:30 - 01:13:36] as an action item and see what representation comes out
[01:13:36 - 01:13:37] from there.
[01:13:41 - 01:13:41] Yeah.
[01:14:00 - 01:14:24] I'm not necessarily like depends how so you can you can imagine a scenario where there
[01:14:24 - 01:14:30] is not necessarily contract and you can imagine a scenario where there is what I can do is
[01:14:30 - 01:14:32] I'll create examples of what I mean by this.
[01:14:32 - 01:14:35] It's very hard to understand this abstractly,
[01:14:35 - 01:14:38] but for example, on the classical side,
[01:14:38 - 01:14:41] the compiler understands the concept of thread blocks
[01:14:41 - 01:14:43] and grids and whatnot, the work items
[01:14:43 - 01:14:45] and how to allocate shared memory
[01:14:45 - 01:14:48] and how to pass that pointer around.
[01:14:48 - 01:14:51] And then the runtime is the one that's doing
[01:14:51 - 01:14:54] the runtime setup for it.
[01:14:54 - 01:14:58] Like I'm launching with 1024 grid size
[01:14:58 - 01:15:00] and two minutes.
[01:15:00 - 01:15:05] a byte of shared memory and this stream. The stream concept doesn't know anything about,
[01:15:05 - 01:15:09] the compiler doesn't know anything about, because that's just establishing the DAG.
[01:15:09 - 01:15:14] Right. But the rest of it, the compiler needs that information in some regards.
[01:15:15 - 01:15:21] We can imagine a scenario where it's separated, where that information can entirely be extracted
[01:15:21 - 01:15:29] from hardware interface, and the runtime only handles the stream or concurrency aspect of
[01:15:29 - 01:15:35] or building the DAG and you can imagine a scenario where the runtime influences that
[01:15:35 - 01:15:42] and the compiler uses the runtime instead or uses a logical representation and resolves that
[01:15:43 - 01:15:52] at a later time. Sorry, I might need to like write a little bit of thing for this. This is
[01:15:52 - 01:15:59] This is potentially hard to explain.
[01:16:13 - 01:16:16] It understands the allocation of like,
[01:16:16 - 01:16:19] memory in the GPU, hard block.
[01:16:28 - 01:16:29] Correct, yeah.
[01:16:29 - 01:16:35] But how to carve them out, or how to like,
[01:16:35 - 01:16:38] how to carve them out, or how to launch specific amount of them.
[01:16:38 - 01:16:41] But that's just a decision that NVIDIA took.
[01:16:41 - 01:16:44] That's not necessarily the only way to design this.
[01:16:44 - 01:16:53] you could separate the two concepts, especially if there's always a static way to know these things.
[01:17:02 - 01:17:12] Yeah, yeah, yeah, yeah. I can write something out for that specific part.
[01:17:12 - 01:17:20] Yeah, it will be clearer if I write out some examples of how things get wrapped.
[01:17:20 - 01:17:26] And this is driven from a very Nvidia specific decision of what a runtime allows you to do
[01:17:26 - 01:17:29] both static and dynamic allocations of shared memory.
[01:17:29 - 01:17:37] It allows you to do runtime figuring out how much grid size to launch because it's dependent
[01:17:37 - 01:17:43] on the problem shapes and sizes that you're handling, like gem problems influences the
[01:17:43 - 01:17:51] grid size. So I don't know what the relative thing for quantum is. Like I know qubits are
[01:17:51 - 01:18:01] known, but a can a application have larger than physical qubits such that it can express
[01:18:01 - 01:18:08] larger problems than what the hard work is schedule possibly not yeah yeah
[01:18:08 - 01:18:14] that's what I'm saying yeah to the stress nice night yeah yeah so so that's kind
[01:18:14 - 01:18:18] of what I'm saying that that is a very like classical versus quantum difference
[01:18:18 - 01:18:24] that is just not there and we don't have to follow what invidiated for the
[01:18:24 - 01:18:31] classical runtime just because it doesn't make sense here but in in on the classical
[01:18:31 - 01:18:36] side, sorry, on the classical side, it does make sense because you can launch more logical
[01:18:36 - 01:18:42] work than the physical hardware. And that's just just do it in iterative space. You, you
[01:18:42 - 01:18:48] do as much work as you have hardware available in one iteration. And once that's done, you
[01:18:48 - 01:18:54] do the next bit and then that's next next next logical work next logical work. And then
[01:18:54 - 01:18:56] you know, eventually you're done with all of the work.
[01:18:56 - 01:18:57] That is not the case.
[01:18:57 - 01:18:58] Right.
[01:18:58 - 01:19:09] And on the quantum system, you could do this your Hamiltonian is sort of block diagonal
[01:19:09 - 01:19:15] in the form, but that means you essentially have separate quantum problems.
[01:19:15 - 01:19:16] Yeah.
[01:19:16 - 01:19:17] Yeah.
[01:19:17 - 01:19:23] In this case, it's the same kernel representing the logical space, but I don't think that's
[01:19:23 - 01:19:37] to for one. But I mean that is the case where you could run sub problems on separate systems
[01:19:37 - 01:19:46] if they're independently there's no correlation. But that is not a full coupled quantum circuit.
[01:19:46 - 01:19:50] Yeah, that makes sense.
[01:20:00 - 01:20:22] Yeah. So I do think there are similarities even there too because I'll give you an example
[01:20:22 - 01:20:29] of similarity. There is a notion in classical computing where you have, it's driven through
[01:20:29 - 01:20:37] different justification. So there's a notion in classical computing where you
[01:20:37 - 01:20:42] want to be able to transfer intermediates between compute units. So like let's say
[01:20:42 - 01:20:47] I loaded up a tile and then I want to share my tile to my neighboring compute
[01:20:47 - 01:20:52] unit. But the only way you can share that or you can use that feature is when you
[01:20:52 - 01:20:57] can guarantee concurrency. You can guarantee two work groups resident on
[01:20:57 - 01:21:00] on neighboring compute units at the same time.
[01:21:00 - 01:21:03] And if that concurrency guarantee cannot be established,
[01:21:03 - 01:21:06] then you just cannot use that ability.
[01:21:06 - 01:21:09] You cannot use that feature.
[01:21:09 - 01:21:13] So that means the logical grid size needs to match
[01:21:13 - 01:21:17] the physical compute unit size, number of compute units.
[01:21:17 - 01:21:21] Otherwise, the circuit, so to say, like bricks,
[01:21:21 - 01:21:23] like you cannot use that at all,
[01:21:23 - 01:21:26] because the guarantee is not there.
[01:21:26 - 01:21:29] I do have to draw for another meeting.
[01:21:29 - 01:21:34] I'll catch up with the recording if there is more discussion.
[01:21:34 - 01:21:39] I also have to leave.
[01:21:39 - 01:21:42] I quickly wanted to mention this.
[01:21:42 - 01:21:45] I think we are losing ourselves in detail in discussion.
[01:21:45 - 01:21:48] Maybe what should we take away from this?
[01:21:48 - 01:21:53] Because I think we are at a point where we can stop this now.
[01:21:53 - 01:21:57] Maybe we should try to like time box our next meeting a bit better
[01:22:23 - 01:22:36] I think that we can take this in both directions. I think for me personally, I think the application
[01:22:36 - 01:22:44] facing one is definitely more than what stuff I'm familiar with, also what I'm working on.
[01:22:44 - 01:22:49] But I guess this should not stop us from considering the low level runtime. But again, I have
[01:22:49 - 01:22:54] this fear, this mind, interfere with other working groups and also with what vendors are
[01:22:54 - 01:23:03] doing where we then don't really have the power to intercheck something I guess.
[01:23:03 - 01:23:29] I think, so I agree with Philip and I mean also my heart is more on the application interface
[01:23:29 - 01:23:39] But I think it might be beneficial for us to start a document where we sketch out what
[01:23:39 - 01:23:47] are the, what we believe are the requirements at these two interface levels.
[01:23:47 - 01:23:58] so that we have some place from where we actually can have a more focused discussion
[01:23:58 - 01:24:00] without losing ourselves in the weeds.
[01:24:17 - 01:24:24] Yes, that's essentially the...
[01:24:47 - 01:24:54] Okay.
[01:25:00 - 01:25:16] Now, I was just saying could be continued the discussion like a saying like on stack
[01:25:16 - 01:25:28] because there might be things that may be informal for a document and yeah like this.
[01:25:28 - 01:25:35] Oh, of course.
[01:25:35 - 01:25:43] Anyway, I'm going to talk now.
[01:25:43 - 01:25:44] Yeah.
[01:25:44 - 01:25:46] I have to.
[01:25:46 - 01:25:47] Thank you.
[01:25:47 - 01:25:48] Thank you.
[01:25:48 - 01:25:49] Okay.
[01:25:49 - 01:25:50] Okay.
[01:25:50 - 01:25:51] Yeah, bye.
