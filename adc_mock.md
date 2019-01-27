# Mock the Administrative Data Centre (ADC) locally

The ADC is the sensitive data store owned by the CSO. It receives source files containing personal data, and outputs anonymised data sets for consumption by the rest of the CSO (and other government departments.)

## Google Cloud Components and GSUtil

Authenticate with google, access the corresponding data project, then view and manipulate files.

```
> gcloud auth login
> gcloud config set project ail-blockathon-portal
> gsutil ls gs://ail-blockathon-portal.appspot.com/sources
```

## Detect received source files and verify their timestamps

The service machine monitoring uploaded files requires an opentimestamps client library. For simplicity, here are the install steps for the python opentimestamps library compatible on most architectures.

```
pip3 install opentimestamps-client
```

Usage for stamping
```
> ots stamp <filename>
```

Usage for verifying
```
> ots verify <filename>
```

Set up a cronjob to repeatedly verify that each stamp file corresponds to the source file submitted.
