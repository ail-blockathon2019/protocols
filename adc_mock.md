# Mock the Administrative Data Centre (ADC) locally

The ADC is the sensitive data store owned by the CSO. It receives source files containing personal data, and outputs anonymised data sets for consumption by the rest of the CSO (and other government departments.)

## Google Cloud Components and GSUtil

Authenticate with google, access the corresponding data project, then view and manipulate files.

```
> gcloud auth login
> gcloud config set project ail-blockathon-portal
> gsutil ls gs://ail-blockathon-portal.appspot.com/sources
```
