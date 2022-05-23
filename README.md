# FHIR-Drill
Fast Healthcare Interoperability Resources (FHIR) is a standard for sharing healthcare information across sites using modern web technologies. While it isn't currently designed for use in scientific research, it has been designed to be very flexible. It is also under continuous refinement and will likely support research needs "out of the box" in the relatively near future. 

For the purposes of this drill, we'll provide 2 FHIR Bundles. These bundles are the output of 2 separate FHIR queries using the standard REST API and contain information about 9 fictitious individuals. To provide a bit of insight into these resources, there are a few things that you will need to understand. 

## Codes are Fundamental
One of the central tenets of interoperability is a common languages, and for this, FHIR uses something called "Code Systems". A CodeSystem is a resource that enumerates each entry in it's vocabulary using "Code", "Display" and "System". The code is the literal coded value associated with the term. The display is "human readable" representation of that code. Think of it as what you would display to the end user. The system is a unique identifier describing the vocabulary itself. For a single CodeSystem with many, many terms, each of them would have the same "system". 

## Identifiers
An identifier in FHIR is used to annotate a resource with some known numeric or alphanumeric string value. To facilitate IDs that may overlap, each identifier can have a system property which can designate the source for the ID. For instance, if we have subject IDs that are unique to the study, the system for that identifier would be a URI representing the study itself. The identifier itself will be found inside the "value" property. 

Because most resources can be known to different entities with different identities, most identifiers are actually lists of identifier resources. 

## Bundles 
Unless a query is highly specific, such as querying for a specific resource by id or it's unique URI, queries from one of the RESTful API calls will result in a Bundle. Bundles are generally simple containers that can provide paging when a given query exceeds the expected resource limit. For the purposes of this test, it is safe to assume that all resources of interest are present in the bundle. 

There are a few key properties of interest for this particular task:
* total -- This indicates the number of resources contained within the bundle
* entry -- This is a list of individual resources

Each entry from the bundle's entry list is wrapped up inside an package which contains a handful of properties, but we only care about one of them:
* resource -- This contains the actual resource of interest. 

For the purposes of this test, each of the JSON files will contain only one Bundle. Each of those bundles will contain several distinct resource objects. 

## Patients
FHIR was designed for interchanging electronic healthcare records, of which, the Patient is a fundamental key. In FHIR, a Patient resource describes a patient of some kind. This is resource where typical "demographic" information will be recorded. Date of Birth, Gender, name, etc are all standard features of the FHIR Patient resource. Because of the special relationship between researchers and the research subjects, most of the typical information that might go inside a Patient record is intentionally unavailable to the researcher. So, for research representation, date-of-birth, name, birthplace, and most other identifiable information will never be found in resources representing conventional resources. 

In addition to the sparse nature of our Patient records, while most people think of patients being human, there are needs to share information about non-human patients as well. As such, things like Race and Ethnicity which don't directly apply to horses and other non-humans and are required to be added as "Extensions". These will all be child resources of the Patient's "extension" property. 

Extensions can be quite verbose and most of the content isn't important for the purposes of this test. What you'll want to capture is the human readable representation, which can be found in a couple of different places for a given extension object. 

Please note that if a patient's race isn't present in the data, the extension property will not be present in the patient resource. 

Key fields of interest:
* id -- This is the internal ID, specific to the FHIR server where the record was generated. This ID will ONLY represent this particular patient ONLY in the given server. In otherwords, this ID is meaningless outside the FHIR server where the record was pulled. This is used by conditions to reference the patient for whom the condition is describing. 
* gender  -- This is really only intended to capture administrative gender, which is very basic.
* identifier -- This indicates the unique ID associated to with the resource itself
* extension -- This just points to a list of extensions. 

## Conditions
Condition resources are intended to represent medical conditions, problems and diagnoses. For the purposes of this test, they represent phenotypic abnormalities. 

A Condition has a small number of key properties:
* code -- The code property in a condition contains a list of "codings". For the purposes of this test, only the code and display properties of the individual "codings" will be important. (see above about purpose of the Code and Display properties)
* verificationStatus -- This points to a list of codes which ultimate have two possible meanings: 
  * Confirmed -- Which indicates that the phenotype described by the condition's code has been observed
  * Refuted -- Which indicates that the phenotype described by the condition's code has been observed to be absent 
* identifier -- This indicates the unique ID associated to with the resource itself
* subject -- This is a reference back to the patient for whom the condition is associated. Please note that the form for these references will always be: Patient/XXXXXXX where XXXXXX is the actual ID associated with the patient. 

# Environment Expectations
Before we get to the challenge itself, we have a few high-level expectations we require out of any response.

## 1. Programming Language
You are free to use whatever language you wish. 

## 2. Executing code / demonstrating work
Given the importance of being able to see your work, the solution provided should include both the code and
a reproducible way for us to view the results. This might be anything from a docker compose file to an
R Shiny application. 

## 3.  Code Style / Format Standards
While we do not have a specific code format / style standard we wish for you to adhere to, you are require to adhere
to _some_ standard.  This must be clearly described in your final pull request.

The _only_ exception to the above is if the language you are either requested to use or chose to use provides a
formatter as a 1st class citizen of the language, e.g. [Golang](https://golang.org/)'s 
[gofmt](https://golang.org/cmd/gofmt/) tool.  In these situations, you _must_ use the language's provided formatter.

# The Challenge
The goal of this challenge is to parse the two bundles and provide a list of patients. Allow the user to select a patient to see details about the sex and race (if available) as well as all phenotypes observed and absent. Also, provide a way to "search" for patients with phenotypes (display and code) that match a pattern provided by the user (i.e. a search box).

[Patient Bundle](https://github.com/torstees/FHIR-Drill/blob/main/resources/Fake_CMG-Patient.json) contains 9 patient resources. 

[Condition Bundle](https://github.com/torstees/FHIR-Drill/blob/main/resources/Fake_CMG-Condition.json) contains 64 condition resources related to the 9 patients above. 

Each Condition resource should have a "subject" which represents one of the 9 patients. 

