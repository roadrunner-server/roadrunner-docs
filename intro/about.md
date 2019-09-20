# What is it?
This is RoadRunner. RoadRunner is infrastructure level framework for your PHP applications written in Golang. It runs
your application in the form of workers.

## Golang
On Golang end RoadRunner runs your PHP application on goroutine and balances the incoming payloads between multiple workers.

![Base Diagram](https://user-images.githubusercontent.com/796136/65347341-79dd8600-dbe7-11e9-9621-1c5f2ef929e6.png)

The data can be coming from the HTTP request, queue or any other way. 

# PHP
RoadRunner keeps PHP worker alive between incoming requests. It means that you can completely eliminate bootload time and
speed up a heavy application by a lot. Since worker locates in resident memory all the open resources will remain open. 
Using Goridge RPC you can quickly offload some of the complex computations to the application server. For example schedule 
background PHP job.

![Base Diagram](https://user-images.githubusercontent.com/796136/65348057-00df2e00-dbe9-11e9-9173-f0bd4269c101.png)