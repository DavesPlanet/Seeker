# Seeker
## integrated tracking of exploited children

## Design Notes:

Seeker is composed of a series of distinct standalone apps. Each app has a specific job to do. This aggregate style architecture has proven very effective at keeping code complexity to a minimum while providing maximum uptime.

Seeker will use the following technologies:

Groovy programming language - similar to Java but on steroids. Pure Java is almost 100% Groovy compatable code but Groovy offers you so much more. Groovy is not as performant as Java, and Java is not as performant as C/C++. The heavy lifting we will do for image processing, deep learning neural networks, and other high performance tasks are done in third party libraries like OpenCV or Encog which are written in very high performance low level languages. 

Spring Boot - each individual app will launch and run quickly and cleanly by itself, no application servers needed. Unit testing and deployment are greatly enhanced with Spring Boot.

MongoDB - a NoSQL style document database with full enterprise support of large datasets. This style of database is the best fit for the types of data Seeker will generate. It particularly excells at handling the massive numbers of tags and sometimes custom data that make up the searchable web we are building. This database will be the heart of Seeker, each individual app will communicate with this shared database.
All Mongo records are required by the engine to have an _id field. Default generated _id fields contain a timestamp.
Mongo does not have tables, it has collections. Because this is not a relational database you can not query across multiple collections.
Each Mongo document has a 16 meg limit

The data structure of objects in Mongo will look something like this:
Database name: Seeker

# Collection:Site - information about each site on the internet that is of interest.
* domainName
*	notes
*	lastSearched
*	searchFrequency
*	referer - how did we learn about this site
*	excludeLinks - array of URL and link text patterns to avoid following, for instance, duplicate content at lower resolution, or text content
*	contentGroup - array of patterns that describe similar content, such as a gallary of one person
*	userId
*	password
*	loginUrl
*	throttleSpeed - default = 10s, if we wish to limit how fast we crawl, actual throttle will be (t / 2) + rnd(t) for a random throttle centered at t

## Collection:Link - links to all pages and content within a site
*	siteId	- the _id from Site
*	url
*	type (page, file type)
*	lastSearched
*	searchFrequency
*	referer - which page sent us here

## Collection:Image
*	linkId - the _id from Link, if that is where the image came from
*	userId - the _id of a user if the image was manually uploaded
*	image

## Collection:Artifact
* imageId - the id of the image this was parsed from
* image - the fragement from the original image representing the artifact
* type - the type of the artifact, such as Face, Ear, Eye, Tattoo, bellybutton, bicycle, whatever
* similarArtifacts - an array of _id from Artifact that have been identified as similar. This array will also contain information about who made the link, when it was made, and how highly scored the link is.

## Collection:Profile
*	images - array of _id from Image related to this profile
* artifacts - array of _id from Artifact related to this profile
* notations - array of notes from users and automated processes
	
Applications
The following applications are needed

## SimilarImage - used to locate duplicates. There is an existing project that does this.

## Web Crawler
Responsible for reading and writing to the Site, Link, and Image document collections as it crawls the internet for data about exploited children. The crawler will stay within the list of Sites that have been identified as appropriate targets. Crawler will add to the list of Sites as it discovers outbound links but those links will be validated and configured before letting Crawler lose on them. The manual oversite of Crawler is to prevent it from crawling and downloading from sites unrelated to our mission. Once a site is identified and setup Crawler will continue to come back to it on a scheduled basis looking for additional information.

## Image Recognition
Image recognition will look through the raw images saved by Crawler in the Image collection and will parse out all manner of useful objects into smaller image fragements. Using deep learning convolutional neural networks it will parse out faces, eyes, ears, birthmarks, tattoos, and a great variety of identifiable artifacts such as pictures, artwork, jewelery, watches, neckties, clocks, eyeglasses. These image fragments will all be processed into the Artifacts collection, tagged with what type of artifact the image represents.

## Face Matching
Face Matching will look through newly discovered Artifact items of type = FACE and will match them against all other Artifact of that type.

## Ear Matching
Ear Matching will look through newly discovered Artifact items of type = EAR and will match them against all other Artifact of that type.

## Eye Matching
Eye Matching will look through newly discovered Artifact items of type = EYE and will match them against all other Artifact of that type.

## Birthmark Matching
Birthmark Matching will look through newly discovered Artifact items of type = BIRTHMARK and will match them against all other Artifact of that type.

## Tattoo Matching
Tattoo Matching will look through newly discovered Artifact items of type = TATTOO and will match them against all other Artifact of that type.

## Other Image Matching
Other Image Matching will look through newly discovered Artifact items that do not have special purpose matchers and will match them against all other Artifact items of the same type.
