# hackair-data-retrieval
Contains components for air quality data collection, image collection from Flickr and web cams, and image analysis for sky detection and localization.

Contains components for air quality data collection, image collection from Flickr and web cams, and image analysis for sky detection and localization.

## Air quality data collector from open sources

Two sources are involved: a) OpenAQ  platform and b) luftdaten

### OpenAQ

#### Description
OpenAQ (https://openaq.org/) is an open data platform that aggregates and shares air quality data from multiple official sources around the world. The data offered by the platform is of high quality as they mainly come from official, usually government-level organizations. The platform offers the data as they are received from their originating sources, without performing any kind of transformations. 

The OpenAQ system checks each data source for updates information every 10 minutes. In most cases, the data source is the European Environmental Agency (EEA) but additional official-level data sources are included (e.g. DEFRA in the United Kingdom).  

The */latest* endpoint (https://docs.openaq.org/#api-Latest) of the API is used, which provides the latest value of each available parameter (pollutant, i.e. NO2, PM10, PM2.5, SO2, CO, O3, BC) for every location in the system. The service receives as parameters, the pollutant and the region (which can be defined either as country name, city or by using coordinates)

### Luftdaten

#### Description
Luftdaten (http://luftdaten.info/) is another source of air quality measurements. It offers data coming from  from low-cost sensors. The Luftdaten API can be accessed by following the instruction in https://github.com/opendata-stuttgart/meta/wiki/APIs.

The data are organized by the OK Lab Stuttgart which is dedicated to the fine dust measurement through the Citizen Science project luftdaten.info. The measurements are provided by citizens that install self-built sensors on the outside their home. Then, Luftdaten.info generates a continuously updated particular matter map from the transmitted data.

The measurements from both sources are stored in a MongoDB.

## Image collection from Flickr and web cams

 - The Flickr collector retrieves the URLs and necessary metadata of images captured and recently (within the last 24 hours) around the locations of interest. This information is retrieved by periodically calling the Flickr API. The metadata of each image is stored in a MongoDB and the URLs are used to download the images and store them until image analysis for supporting air quality estimation is performed.

In order to collect images the flickr.photos.search endpoint was used. For determining the geographical coverage of the query the woe_id parameter was used. This parameter allows geographical queries based on a WOEID (Where on Earth Identifier), a 32-bit identifier that uniquely identifies spatial entities and is assigned by Flickr to all geotagged images. Furthermore, in order to retrieve only photos taken within the last 24 hours, the min/max_date_taken parameters of the flickr.photos.search endpoint are used. These parameters operate on Flickr’s ‘taken’ date field which is extracted, if available, from the image’s Exif metadata. However, the value of this field is not always accurate as explained in Flickr API’s documentation.
 
An idiosyncrasy of the Flickr API that should be considered is that whenever the number of results for any given search query is larger than 4,000, only the pages corresponding to the first 4,000 results will contain unique images and subsequent pages will contain duplicates of the first 4,000 results. To tackle this issue, a recursive algorithm was implemented, that splits the query’s date taken interval in two or more and creates new queries that are submitted to the API. This mechanism offers robustness against data bursts.


 - Webcams.travel (https://developers.webcams.travel/) is a very large outdoor webcams directory that currently contains 64,475 landscape webcams worldwide. Webcams.travel provides access to webcam data through a comprehensive and well-documented free API. The provided API is RESTful, i.e. the request format is REST and the responses are formatted in JSON and is available only via Mashape (https://www.mashape.com/). The collector implemented uses the webcams.travel API to collect data from European webcams. The endpoint exploited  is the /webcams/list/ and apart from the continent modifier that narrows down the complete list of webcams to contain only webcams from specific continent, two other modifiers are used: a) orderby and b) limit. The orderby modifier has the purpose of enforcing an explicit ordering of the returned webcams in order to ensure as possible that the same webcams. The limit modifier is used to slice the list of webcams by limit and offset given that the maximum number of results that can be returned with a single query is 50. 




## Image analysis for sky detection and localization
Image Analysis involves all the operations required for the extraction of Red/Green (R/G) and Green/Blue (G/B) ratios from sky-depicting images. The service accepts, carries out all the required processing, and responds with the results of the image analysis. The service accepts HTTP post requests that specify either a
set of local (to the server that runs the service) paths that correspond to images already downloaded on the server
through one of the image collectors (Flickr or webcams) or a set of image URLs36. 

In the latter case, all images are first
downloaded (using a multi-threaded image download implementation).
The IA service uses internally three components, each one implementing a processing step of the image analysis
pipeline, i.e. concept detection, sky localization and ratio computation. When an IA request is received, the IA service
first sends a request to the concept detection (CD) component (step 1) which implements the concept detection
framework described in D3.1. The CD component applies concept detection on each image of the request and returns
a set of scores that represent the algorithm’s confidence that the sky concept appears in each image. When a response
is received by the CD component, the IA service parses it to check which images are the most likely to depict sky based
on the confidence scores calculated by the CD component (step 2). A relatively high (0.8) threshold is used to lower
the probability of sending non-sky-depicting images for subsequent analysis. At step 3, the IA service sends a request
to the sky localization (SL) component which implements the FCN-based sky localization framework described in D3.1.
This is a computationally heavy processing step that is carried out on the GPU of the IA server. The response of the SL
component is the sky mask of each image of the request. To minimize the time required for sending the masks to the
IA service, a compression algorithm is first applied to reduce the size of the masks. Then, the IA service receives the
response from the SL component (step 4) and sends a request to the ratio computation (RC) component (step 5). The
RC component takes the sky masks computed by the FCN approach as input, refines them by applying the heuristic
approach on top of them (see section 3.1) and computes the R/G and G/B ratios of each image (in case all checks of
the heuristic approach are successfully passed). Finally, the IA service parses the response of the RC component (step
6) and combines the results of all processing steps to synthesize the IA response (step 7).


nohup python TF_detection_service.py > detection_log.txt 2>&1




# hackAIR Decision Support API


## Description
The hackAIR Decision Support (DS) API is a dedicated software responsible for: (i) the representation of a problem (request) for decision support in a formal, comprehensible and hackAIR-ontology-compatible way; (ii) the communication between the <a href="https://platform.hackair.eu/" target="_blank">hackAIR UI (app/platform)</a>, or even other third-party DS systems, and the <a href="https://mklab.iti.gr/results/hackair-ontologies/" target="_blank">ontology-based representation and reasoning knowledge base (KB)</a>, which supports the recommendation mechanism. The involved web-services were created with the adoption of state-of-the-art technologies: RESTful communication, exchange of information on the basis of JSON objects, etc. The hackAIR DS API is publicly available and may run both as an independent service or as an integrated service on the hackAIR app/platform. 


## Web-Services
Up to now, the hackAIR DS API offers the following web services through POST requests:
* _{BASE_URL}/hackAIR_project/api/dynamicPopulation_: performs the dynamic population of involved data (user profile and enviromnental data) in the hackAIR KB for further manipulation.
* _{BASE_URL}/hackAIR_project/api/requestRecommendation_: performs a step-by-step process, i.e. (i) receives a JSON object in pre-defined format, through a POST request to the service of discourse, (ii) converts the JSON data to a hackAIR-compatible ontology-based problem description language for populating new instances (user profile details and environmental related data) in the knowledge base; (iii) triggers the hackAIR reasoning mechanism for handling the available data and rules and for inferencing new knowledge, i.e. provide relevant recommendations to the users. 


### Key features 
The hackAIR DS module supports:
* Multi-threading requests (syncronized, i.e. first come first served). 
* Combined user-profiles' (primary and secondary) requests for decision support.
* Recommendation messages in three different languages: English, German and Norwegian


### JSON parameters
Below, we specify all the mandatory and optional JSON parameters that are accepted in the POST request:

Parameter | JSON Type | Mandatory(M) / Optional(O) | Accepted values
:--- | :---: | :---: | :---
`username` | object | M | any *string* value
`gender` | object | O | One of the following: *male*, *female*, *other*
`age` | object | M | any *integer* value
`locationCity` | object | M | any *string* value
`locationCountry` | object | M | any *string* value
`isPregnant` | object | O | any *boolean* value
`isSensitiveTo` | array | O | One or more of the following: *Asthma*, *Allergy*, *Cardiovascular*, *GeneralHealthProblem*
`isOutdoorJobUser` | object | O | any *boolean* value
`preferredActivities` | object | O | `preferredOutdoorActivities`
`preferredOutdoorActivities` | array | O | One or more of the following: *picnic*, *running*, *walking*, *outdoor job*, *biking*, *playing in park*, *general activity*
`airPollutant` | object | M | Both: `airPollutantName`, `airPollutantValue`
`airPollutantName` | object | M | One of the following: *PM_AOD*, *PM10*, *PM2_5*, *PM_fused*
`airPollutantValue` | object | M | any *double* value
`preferredLanguageCode` | object | O | One of the following: *en*, *de*, *no*
`relatedProfiles` | array | O | One or more JSON objects, each of which includes the aforementioned mandatory/optional fields.


### Example JSON object

#### With primary and secondary profile description, in one single request

```
{
  "username": "Helen_Hall",
  "age":"32", 
  "locationCity": "Berlin",
  "locationCountry": "Germany",
  "isPregnant": false,
  "isSensitiveTo": ["Asthma"],
  "preferredLanguageCode": "de",
  "airPollutant": {
    "airPollutantName": "PM_fused",
    "airPollutantValue": "3.5",
  }
  "preferredActivities": {
    "preferredOutdoorActivities": ["picnic","running"]
  },
  "relatedProfiles": [{
    "username": "Helen_Hall_secondary_profile",
    "gender":"female",
    "age":"1", 
    "locationCity": "Berlin",
    "locationCountry": "Germany",
    "preferredLanguageCode": "de",
    "airPollutant": {
      "airPollutantName": "PM_fused",
      "airPollutantValue": "3.5"
    }
  }]
}
```


## Requirements - Dependencies
The hackAIR DS API is implemented in [Java EE 7](https://docs.oracle.com/javaee/7/index.html) with the adoption of [JAX-RS](http://docs.oracle.com/javaee/6/api/javax/ws/rs/package-summary.html) library. Additional dependencies are listed below:
* [Apache Jena](https://jena.apache.org/): a free and open-source Java framework for building Semantic Web and Linked Data applications.
* [SPIN API](http://topbraid.org/spin/api/): an open source Java API to enable the adoption of SPIN rules and the handling of the implemented rule-based reasoning mechanism. 
* [GlassFish Server 4.1.1](http://www.oracle.com/technetwork/middleware/glassfish/overview/index.html): an open-source application server for the Java EE platform, utilised for handling HTTP queries to the RESTful API.
* [json-simple](https://github.com/fangyidong/json-simple): a well-known java toolkit for parsing (encoding/decoding) JSON text.
* [hackAIR Knowledge Base (KB) and Reasoning Framework](https://mklab.iti.gr/results/hackair-ontologies/): this regards the implemented ontological representation of the domain of discourse that handles both the semantic integration and reasoning of environmental and user-specific data, in order to provide recommendations to the hackAIR users, with respect to: (i) personal health and user preferences (activities, daily routine, etc.), and (ii) current AQ conditions of the location of interest. The hackAIR DS module utilises the sources of the hackAIR KB and reasoning framework as a background resource of information, from which it acquires the necessary semantic relations and information in order to support relevant recommendations’ provision to the users upon request for decision support. 


## Instructions
1. Install Java EE 7 and GlassFish 4.1.1 in your computer.
2. Clone the project locally in your computer.
3. Run Glassfish server and deploy [hackAIR_project.war](hackAIR_project/target) application.
4. Submit POST requests in relevant web-services, as described [here](https://github.com/MKLab-ITI/hackair-decision-support-api#web-services)

or

1. Install Java EE 7 and a common Java IDE framework.
2. Clone the project locally in your computer.
3. Import the java project to the workspace of the IDE framework.
4. Set up a Glassfish server from the IDE environment to run locally.
5. Run the project through the IDE utilities.
6. Submit POST requests in relevant web-services, as described [here](https://github.com/MKLab-ITI/hackair-decision-support-api#web-services)


## Resources
The official hackAIR ontology resources are available [here](https://mklab.iti.gr/results/hackair-ontologies/).


## Citation
Riga M., Kontopoulos E., Karatzas K., Vrochidis S. and Kompatsiaris I. (2018), An Ontology-based Decision Support Framework for Personalised Quality of Life Recommendations. In: Dargam F., Delias P., Linden I., Mareschal B. (eds) Decision Support Systems VIII: Sustainable Data-Driven and Evidence-Based Decision Support. 4th International Conference on Decision Support System Technology (ICDSST 2018). Lecture Notes in Business Information Processing (LNBIP), Volume 313, Springer, Cham. doi: [https://doi.org/10.1007/978-3-319-90315-6_4](https://doi.org/10.1007/978-3-319-90315-6_4).


## Contact
For further details, please contact Marina Riga (mriga@iti.gr)


## Credits
The hackAIR Decision Support API was created by <a href="http://mklab.iti.gr/" target="_blank">MKLab group</a> under the scope of <a href="http://www.hackair.eu/" target="_blank">hackAIR</a> EU Horizon 2020 Project.



