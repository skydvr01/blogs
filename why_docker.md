## Docker, hek?!
[Docker](https://www.docker.com/) is a software that helps to create a single REPEATABLE deployable unit (also called a "container") for applications no matter where the container is deployed. With a programtic interface, Docker can also help create a scalable development and deployment process that maintains the same code (or binary), runtime, system tools and system libraries across development and production servers.

This concept of a container has been revolutionary for both software and hardware because developers can now ensure a consistent software driven deployment process to production servers as well as reducing the need for headcount specifically dedicated to server deployment and maintanence. For those of you working with hardware, Raspberry Pis also now support Docker:
[https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/](https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/)

"Docker is an application delivery technology, not a virtualization technology..." This was the key statement in the [Docker blog](https://blog.docker.com/2016/03/containers-are-not-vms/) that helped me understand what Docker is, how it is different from virtual machines and why Docker is important. 

Here are some additional links that can help you understand the benefits of Docker beyond simply being a requirement for the MobiledgeX platform:
* https://www.capside.com/labs/using-docker-for-development/ 
* https://codeburst.io/the-advantages-of-using-docker-for-web-development-23096c457fad

## Benefits of Docker During Development
Some of the benefits of using Docker in development include:

- Reproducibility -- Faster development machine setup for new developer onboarding and a standard development environment used by all team members
- Environment Management -- Updating dependencies centrally and using the same container everywhere,
- Continuous Integration -- Digital and software based approach to development and deployment 
- Security -- If one container is compromised the others remain unaffected.

From the development standpoint, Docker has been amazing as it has eliminated the need for me to install and manage language specific dependencies. Additionally, I haven't had to deal with any file folder access rights issues that use to be a problem with NodeJS. 

To go from code to Github to live server deployment can be accomplished with a single developer, setup once and future deployments only require a push to Github. Multiple containers can also be loaded into Docker as your needs increase. 

## Next Steps with Docker
Now that we have covered the benefits of Docker, let's move onto code examples
- [Docker NodeJS Hello World](https://github.com/skydvr01/MobiledgeX_Docker_NodeJS_Hello_World)
- [Docker GoLang Hello World](https://github.com/skydvr01/MobiledgeX_Docker_GoLang_Hello_World)



## Reference Blogs and Youtube Videos
- [https://www.youtube.com/watch?v=SC7lLm6QAb8](https://www.youtube.com/watch?v=SC7lLm6QAb8)
- [https://www.capside.com/labs/using-docker-for-development/](https://www.capside.com/labs/using-docker-for-development/)
