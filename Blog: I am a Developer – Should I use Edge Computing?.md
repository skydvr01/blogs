 

Edge computing is a grouped computing model in which computing takes place near the physical location where data is being accumulated and interpreted. This advanced infrastructure gathers information and securely processes data in real-time and on-site, rather than on a centralized server or in the cloud. Edge computing also connects smart and mobile devices to the network, enabling workloads to run closer to the client. Fortunately, MobiledgeX provides edge computing capabilities that manage the deployment and scaling of server applications.

There are high levels of interest in edge computing within the infrastructure and telecoms industry. Still, we don't hear as much talk about edge computing within the application world. Luckily, things are changing. We hope to highlight how edge computing can solve many of the common problems you may encounter within application development.

Many applications today use some form of cloud computing for their backend services, e.g., hosting databases or data analytics. For consumer and business applications, public clouds, such as AWS, Google Cloud, and Microsoft Azure, are being utilized. The primary driver for using public clouds is the ability to quickly access and scale resources globally.

Unfortunately, using traditional cloud computing comes with some disadvantages. These difficulties include data transport costs between the client and the cloud, potential security, and privacy issues. These problems occur regularly with public clouds that connect via the internet. Furthermore, performance issues abound due to the time required for the data transport between the clients and the cloud — this loss of performance, which is referred to as round-trip-time or latency, is the most prominent problem.

Without a solution, most applications do not leverage cloud computing and remain mainly on-device. An example is computer and console-based games, which are very demanding in terms of computing capacity. The high demand produces high-resolution and low input-output latency that delivers an excellent customer experience, but these experiences fall short in mobile devices. Regrettably, this predicament in mobile devices is magnified as the demand for better experiences increases. Devices are constrained in terms of form factor, performance, and price. The average consumer doesn't want to wear bulky augmented reality glasses containing a lot of hardware, batteries, and most importantly, wear something uncomfortable.

One way to overcome some of the problems mentioned is to tether one device to the other. Attaching your AR glasses to a CPU is impractical, it doesn't solve the issue of total system cost, and it also implies some usability constraints. Think cables and portability. Smartphones, with their ability to support the new wave of augmented reality applications, do overcome the cable concerns but still suffer from latency issues.

Currently, Apple and Google are further developing and offering augmented reality enabling frameworks named ARKit and ARCore. ARKit and ARCore frameworks build a 3D model of your environment based on the camera and track the orientation of the device within its model. While it is remarkable what those frameworks can do, they are severely limited by the device's computational ability. For example, you desire to look at a bottle of water through an AR application or pair of AR glasses and have a precise model of that bottle, including its semi-transparency under the given light condition. A virtual object may be placed behind the bottle and is rendered accurately according to this real-world object behavior. However, such a model can only be created using advanced computer vision algorithms utilizing server-grade CPUs and GPUs.

If your application is subject to performance issues, edge computing is the solution for you.

Does your application utilize the public cloud but only offer limited experience and, does your device underperform when connected to a public cloud? Similarly, if you are running applications on the device and are restrained from improving the user experience due to the device's processing constraints, then edge computing is an excellent solution. Edge computing can surmount those problems and create a flawless and more immersive experience.

### What can the edge do for me?

There are many discussions with different interpretations of what or where the edge is. On a high level, edge computing means to perform computing near the origin of the data that is processed. Edge computing can be in-network, on-premise, or on-device. From an application developer's standpoint, edge computing is similar to cloud computing. It is equal in terms of how it integrates into an application architecture but, edge computing possesses specific advantageous properties within the data path between the client and the server. The principal components of edge computing are:

* Low network latency due to the close physical proximity between client and server.
* Promoting low jitter from the data not being routed through the internet, thereby minimizing hops, and avoiding traffic prioritization. (Jitter refers to the variation of latency over time. Low jitter indicates a stable, predictable latency.)

* Data sovereignty as the data path is fully known and managed.

* Location awareness with precision depending on the distribution level of computing and use of tools that assign a client to the nearest edge compute instance.

The quintessential ingredient that makes edge computing useful is its handling of mobility. It is easy to manage this in a static local setup (it is the modern version of a traditional server room). However, many experiences come to life with a mobile approach, which means two things:

* You benefit within your experience wherever you are, independent of a specific connectivity setup.

* Also, if the experience is nomadic, problems are resolved. For example, an automobile, drone, or a person in motion using edge computing is unaware of changes in performance dependent on latency. When in travel, the edge compute will determine and adjust to the nearest edge instance while the client travels. This adjustment allows the distance to reduce, enabling optimal computing.

Accordingly, our focus is on utilizing the mobile network edge via existing 4G LTE networks and future 5G mobile networks. The question of what network technology is required depends entirely on the specific use, its requirements, and how the aspect of latency will differ.

### How edge computing is utilized within an application? The four patterns of edge application architecture

The edge allows workloads with distinct characteristics to run on the network between the client and the server. The application architecture must be adapted to the particular experience to utilize this enhancement.

Improving already existing cloud-based applications is possible by moving the workloads that suffer from the disadvantages of the public cloud, toward the edge. Keep in mind; this may require dividing workloads based on criteria and other commercial considerations. Typically, cases for edge computing aim to improve the user experience and reduce cost. These targets are reflected in the patterns 1 and 2 described below resulting in a 3-layered scenario (device <-> edge <-> public cloud).

Solving device constraints by moving a part of the on-device computation to the edge, referred to as compute offload, requires additional considerations about the connectivity aspect, such as low connectivity scenarios. Changing the existing on-device application architecture in a client-server architecture is necessary.

There are four fundamental patterns to integrate edge computing in the application architecture. The first two focus on the interaction between a device and the edge compute while the last two describe a three-tiered architecture: device<->edge<->public cloud. Any application architecture may use one or multiple patterns in combination for various workloads:

1. **Single user closed-loop edge computing. Used for compute offload.** <add image here> In this scenario, edge computing is used for processing data from the device. Video data is commonly generated with high data traffic, causing significant latency. For example, a video is streamed to the edge compute instance where the data is processed and sent back to the device, giving the device lower latency and jitter. This pattern is also beneficial for more downstream-centric scenarios where a game remotely rendered on the edge allows a low data traffic input event in the upstream. This orientation also provides lower latency and jitter. The underlying drivers are typical device constraints, such as form factor or cost. Using this pattern eliminates the need for building an expensive GPU into the device by leveraging shared GPU resources on the edge.

2. **Multi user communication** This pattern focuses on leveraging the short network route to the edge to establish a group communication mechanism for devices that are in geographical proximity. For example, devices within the same city attached to the corresponding edge compute instance can be very useful in combination with a cellular network. Once combined, these devices are now able to establish reliable and scalable communication in real-time between a more significant number of devices such as AR gaming devices or a swarm of drones. That pattern may combine with computations done on the edge, as shown in pattern 1. <ADD IMAGE HERE> Unfortunately, this is not always the case. The underlying driver is the lack of scalable peer-to-peer communication options for local device groups since Wi-Fi does not provide a reliable service, due to interference in crowded areas.

3. **Upstream Data thinning** This next pattern illustrates optimizing (reducing) data transfer toward a public cloud. This pattern is commonly applied when a large number of devices are generating data with high traffic. For example, consider a number of surveillance cameras that are processed and stored in the public cloud generating high data demand. The edge, positioned between the cloud and the device, lessens the data traffic transferred to the cloud by performing pre-processing of the incoming data. Once processed, the edge prioritizes the data before it is transmitted to the cloud. This case doesn't focus on reducing latency and jitter but supports a lower cost scenario.

<ADD IMAGE HERE>

4. **Downstream personalization** This case is very similar to the traditional CDN approach, which captures content as near to the user as possible. Although this is a valid scenario to support edge computing, there are also new scenarios within this pattern that use a general "master" stream. A master stream is a personalized stream of data for each viewer, such as showing a banner advertisement based on the viewers' preferences. The original rendering is performed on the edge. The minimized cost and improved experience is the motivation for this particular pattern.

<ADD IMAGE HERE>

The data sovereignty aspect supports these four patterns and depends on the use case and relative location. For an application to leverage edge computing, it is essential to consider how to change the application architecture and integrate the patterns as described above. Please note, in most scenarios, low latency is a critical aspect. Low latency implies the application is appropriately optimized. Furthermore, using low-latency transmission protocols between client and server ensures the advantages of edge computing is utilized correctly.

### So, is edge computing for me?

Edge computing is an innovative form of computing that enhances cloud computing with its unique characteristics and is not intended to replace cloud computing. It is a complementary technology that many applications can employ with the cloud. Thus, the question is, not if edge computing is an option for an entire application, but specific components, workloads, and microservices of an application. It is, of course, essential to identify those components and prepare them adequately to deploy to the edge.

Here are a few questions that may help you determine whether edge computing will assist you. Take this as guidance; many more scenarios exist, but don't let this stop your creativity. If you answer yes to any of the questions, edge computing offers the solution you need:

* Does my application have any time-critical components?

* Does my current client application experience limits due to computing capability, memory, or storage available?

* Is my existing cloud application suffering from slow response times due to network delays?

* Is the cost of data transfer between client and server costly?

* Do I hesitate to send my data over the internet to a public cloud?

* Do I have location-dependent data such as a model of the real-world environment and would appreciate the option to offload workloads?

Even more applications can benefit from edge computing. Please see several real-world examples deployed on our platform for further examples: https://mobiledgex.com/use-cases.

 
