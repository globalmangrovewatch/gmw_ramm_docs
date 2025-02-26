# Radar Alerts for Mangrove Monitoring (RAMM)

![company_logos](public/hatfield-esa-wetlandsint-abers.png)

Global Mangrove Watch (GMW) publishes annual global data of the extent of mangrove forests and since 2019 a mangrove deforestation alert system using Copernicus Sentinel-2 imagery (Bunting et al. 2023). The optical image alert system is limited by cloud cover, often delaying the detection of mangrove loss and diminishing its impact for mangrove conservation. Sentinel-1 Synthetic Aperture Radar (SAR) penetrates cloud cover and offers potential to provide consistent monthly alerts. However, in coastal regions analytical methods must address the complex interaction of the SAR signal with mangrove canopy and water.

The consortium combines the expertise, experience, and leadership positions of three organizations – an environmental EO application development company (Hatfield), champion user (DGES/GMW), and early adopter (Wetlands International):
  -	Hatfield provides deep project management expertise, experience in remote sensing and mangrove assessment and monitoring, cloud platform exploitation, and engagement with the user community including IFIs.
  -	DGES is the technical lead for GMW and provides deep scientific and technical expertise in remote sensing, mangrove assessment and monitoring, and computer science with a demonstrated publication and user community engagement track record.
  -	Wetlands International is a global leader in mangrove conservation and restoration and coordinating member of the GMA, with expertise in specification and dissemination of remote sensing products and advocacy on policy.

RAMM is implemented over the <a href="https://www.globalmangrovewatch.org/" target="_blank">Global Mangrove Watch (GMW)</a> baseline extent (currently 2020) in a two stage per pixel approach, designed to maximize efficiency. 
The first stage is a simple rule-based approach based on thresholding of backscatter value and difference from the previous year’s median backscatter value, which reduces effects of seasonal and tidal variability and radar speckle. 
The thresholds are designed to minimize false negatives (i.e. capture all possible mangrove loss).
The second stage is a 1-Dimensional Convolutional Neural Network (1D-CNN) that is trained on a dataset of confirmed alerts and false positives (both produced by GMW).
The integration of false positives into the training dataset encourages the model to recognise and flag falsely identified alerts provided by the rule-based first stage as deep learning algorithms are known to extract fine-grained patterns between domain distributions. 


- Andrew Dean - <a href="https://hatfieldgroup.com" target="_blank">Hatfield Consultants</a>
- Benjamin Smith - <a href="https://hatfieldgroup.com" target="_blank">Hatfield Consultants</a>
- Pete Bunting - <a href="https://www.aber.ac.uk/en/" target="_blank">Aberystwyth University</a>
- Dr. Victor Tang - <a href="https://hatfieldgroup.com" target="_blank">Hatfield Consultants</a>
- Lammert Hildarides - <a href="https://www.wetlands.org" target="_blank">Wetlands International</a>
- Frank Martin Seifert - <a href="https://www.esa.int" target="_blank">European Space Agency (ESA)</a>

---

## Platform

With an Open Call with the European Space Agency (ESA),
funding was provided to develop the RAMM platform on <a href="https://creodias.eu/" target="_blank">CREODIAS</a>.
CREODIAS is a cloud-based platform that provides access to
<a href="https://creodias.eu/eodata/all-sources/" target="_blank">Copernicus data</a> and processing capabilities.
The platform is built on OpenStack technologies and provides
a development environment for users to interact with the data
and processing capabilities.

## Demonstration Areas

The development of RAMM is focused on two demonstration areas: Guinea Bissau and North Kalimantan, Indonesia.


### Guinea Bissau

  
<div
    style="
        justify-content: center;
        display: flex;
        "
>
<img src="public/guinea-bissau-hatfield-map.png" alt="guinea_bissau" width="50%">
</div>
<br />

~7 to 9% of the country's total land and 86% of the
coastline are covered by mangroves. 
As a byproduct of land conversion for rice 
agriculture and coastal development, mangroves are
experiencing loss and degradation.

Guinea Bissau has been identified by GMW as a hot-spot 
for mangrove change in Africa. 

### North Kalimantan, Indonesia

<div
    style="
        justify-content: center;
        display: flex;
        "
>
    <img src="public/north-kalimantan-hatfield-map.png" alt="north_kalimantan" width="50%">
</div>
<br />

High levels of mangrove loss and degradation in the province of North Kalimantan, Indonesia (which is located on the island of Borneo), primarily due to land conversion for agriculture and aquaculture, as well as pollution.​
There is a particular focus of many mangrove restoration initiatives by the Indonesian government and other International Financial Institutions (IFIs).

## Mangrove Loss Alerting Algorithms

To detect mangrove deforestation, RAMM implements a two-stage
approach executed on a recurring basis using Sentinel-1 SAR
data. 
Running the pipeline at the end of each month, the pipeline
iterates through the following stages:

1. **Change Detection**: The first stage is to detect change
   in the SAR data. This is done by gathering the previous
   months acquisitions over the provided target area.
   The acquisitions are stacked and potential mangrove loss pixels are
   identified using a tuned threshold.
   The potential mangrove loss pixels pixels are then compared 
   to the same month of the 
   previous year's median composite and classifying as possible
   alerts if the absolute difference is greater than a threshold.
   Outputs are then written to BLOB Storage. 

2. **Mangrove Alert Validation**: The second stage pulls the 
    first stage outputs from BLOB Storage and
    uses a Convolutional Neural Network (CNN) to validate
    that each alert is a mangrove deforestation event.
    The validated alerts are then written to BLOB Storage 
    as a GeoJSON file that can be ingested by the GMW Portal.

The first stage thresholds are tuned by performing a sensitivity
analysis across real-world data.
The second stage CNN is trained with a dataset of derived alert points produced by GMW using
their latest global mangrove extent map and Sentinel-1 data.

## Platform Architecture

The RAMM platform is built on the following technologies:

- <a href="https://creodias.eu/" target="_blank">CREODIAS</a>: The cloud-based platform that provides access to Copernicus data.
- <a href="https://aws.amazon.com/s3/" target="_blank">S3</a>: The object storage service used to store the pipeline outputs.
- <a href="https://www.openstack.org/" target="_blank">OpenStack</a>: The underlying technology offered by CloudFerro for CREODIAS.
- <a href="https://restfulapi.net/" target="_blank">REST API</a>: The API is used to interact with the platform.
- <a href="https://www.rabbitmq.com/" target="_blank">RabbitMQ</a>: The message broker used to manage the pipeline.
- <a href="https://www.amqp.org/" target="_blank">AMQP</a> is the protocol used by RabbitMQ.
- <a href="https://www.docker.com/" target="_blank">Docker</a>: The containerization technology used to package the pipeline.
- <a href="https://keda.sh/" target="_blank">KEDA</a>: The Kubernetes Event-Driven Autoscaler used to scale the pipeline.

The platform is designed to be scalable and flexible to allow for
the analysis of the entire global mangrove extent.

Developers of the algorithms are provided with a development environment
to interact with the data and processing capabilities of the platform.
They access the platform through a Web UI utilising user authentication. 

The platform is designed to be event-driven, with the pipeline being
triggered by a REST API call that submits a job to populate the
first stage message queue with the required parameters.
At this point, the target area is also distributed into smaller
tiles (MGRS) to allow for parallel processing of the SAR data.
It is this message queue that receives and distributes the
tasks from the first to second stage, ensuring that the pipeline
is scalable, efficient, and concurrent.

<div
    style="
        justify-content: center;
        display: flex;
        "
>
&copy; 2025 Hatfield Consultants
</div>