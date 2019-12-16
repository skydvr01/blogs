Title: How To: Android Workshop: Add Edge Support to Workshop App

##Introduction

This workshop demonstrates how to add Edge support to the Workshop app. We will also introduce you to MobiledgeX's Distributed Matching Engine (DME) APIs. 

We will be making use of a MobiledgeX library: The **MobiledgeX MatchingEngine** library exposes various services that MobiledgeX offers such as finding the nearest MobiledgeX Cloudlet Infrastructure for client-server communication or workload processing offload.

This library has been published to a Maven repository and we will be adding them to our workshop project. To interface with the library, you must update the workshop code to instantiate several classes and implement several interfaces. This document will walk you through how to do this.

##Workshop Goals

*Start with the Workshop Skeleton app, add Edge support to register the client, and find the closest Edge cloudlet.
*Gain understanding of Distributed Matching Engine APIs

##Prerequisites

*Experience with Android app development.
*A current install of Android Studio. This installation can be a lengthy process, therefore, we recommend that you ensure your development environment is stable before you begin the workshop.
*Access to the Workshop Skeleton app code.
*Access to the MobiledgeX maven SDK repository. Go here to create a user login, and validate the email address.
*Note: If you are a hackathon participant, please see the event organizer for your specific user name and password.

##Acquire Android Studio Workshop Project
Navigate to where you want your edge-cloud-sampleapps repo to be located:
