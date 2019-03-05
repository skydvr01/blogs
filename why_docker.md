## Docker, hek?!
[Docker](https://www.docker.com/) is a software that helps to create a single REPEATABLE deployable unit (also called a "container") for applications no matter where the container is deployed. With a programtic interface, Docker can also help create a scalable development and deployment process that maintains the same code (or binary), runtime, system tools and system libraries across development and production servers.

This concept of a container has been revolutionary for both software and hardware because developers can now ensure a consistent software driven deployment process to production servers as well as reducing the need for headcount specifically dedicated to server deployment and maintanence. For those of you working with hardware, Raspberry Pis also now support Docker:
[https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/](https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/)

## Advantages over Virtual Machines
Containers offer similar resource allocation and isolation benefits as virtual machines. However, the similarity ends there.

A virtual machine needs its own guest operating system while a container shares the kernel of the host operating system. This means that containers are much lighter and need fewer resources. A virtual machine is, in essence, an operating system within an operating system. Containers, on the other hand, are just like any other application in the system. Basically, containers need fewer resources (memory, disk space, etc.) than virtual machines, and have much faster start-up times than virtual machines.

## Benefits of Docker During Development
Some of the benefits of using Docker in development include:

- Reproducibility -- Faster development machine setup for new developer onboarding and a standard development environment used by all team members
- Environment Management -- Updating dependencies centrally and using the same container everywhere,
- Continuous Integration -- Digital and software based approach to development and deployment 
- Security -- If one container is compromised the others remain unaffected.

From the development standpoint, Docker has been amazing as it has eliminated the need for me to install and manage language specific dependencies. Additionally, I haven't had to deal with any file folder access rights issues that use to be a problem with NodeJS. 

To go from code to Github to live server deployment can be accomplished with a single developer and with , setup once and future deployments only require a push to Github. 

## Different Docker Usecases 
Continuous Integration and Delivery with Docker. Continuous delivery is all about reducing risk and delivering value faster by producing reliable software in short iterations.

## Reference Blogs and Youtube Videos
- [https://www.youtube.com/watch?v=SC7lLm6QAb8](https://www.youtube.com/watch?v=SC7lLm6QAb8)
- [https://www.capside.com/labs/using-docker-for-development/](https://www.capside.com/labs/using-docker-for-development/)
