# Mock the Administrative Data Centre (ADC) locally

The ADC is the sensitive data store owned by the CSO. It receives source files containing personal data, and outputs anonymised data sets for consumption by the rest of the CSO (and other government departments.)

## Google Cloud Components and GSUtil

Authenticate with google, access the corresponding data project, then view and manipulate files.

```
> gcloud auth login
> gcloud config set project ail-blockathon-portal
> gsutil ls gs://ail-blockathon-portal.appspot.com/sources
```

## Protocol for confirming that what was received was the same file as what was sent

To do this we are going to query whether the file was submitted alongside valid evidence that the hash checksum of the file existed at a particular time. This statement of the file's integrity was submitted to a blockchain third party notary on creation, which makes the verification action more robust (via third party backup) than only storing checksums locally.

*Setup*

The service machine monitoring uploaded files requires an opentimestamps client library. For simplicity, here are the install steps for the python opentimestamps library compatible on most architectures.

```
> pip3 install opentimestamps-client
```

*Monitoring and verification*

Ideally we want to pick up on any file changes and cue a verification action at that point. To keep it simple and straightforward, we can also just iterate through all files available and check that their timestamp file confirms what they claim to be. On a linux machine this would be added to the crontab.

We'll assume a naming convention is imposed such that each file and their timestamp file have the same name with and without the `.ots` suffix.

Mirror the cloud storage locally for demo's sake
```
> gsutil rsync gs://ail-blockathon-portal.appspot.com/sources ./adc_local_mirror
```

For each timestamp file:
```
> ots verify <filename>
```

Expected outputs while the stamp is propagating:
```
Calendar https://bob.btc.calendar.opentimestamps.org: Pending confirmation in Bitcoin blockchain
Calendar https://alice.btc.calendar.opentimestamps.org: Pending confirmation in Bitcoin blockchain
Calendar https://finney.calendar.eternitywall.com: Pending confirmation in Bitcoin blockchain
Calendar https://btc.calendar.catallaxy.com: Pending confirmation in Bitcoin blockchain
```

Expected successful output:
```
Got 1 attestation(s) from https://btc.calendar.catallaxy.com
Got 1 attestation(s) from https://alice.btc.calendar.opentimestamps.org
Got 1 attestation(s) from https://finney.calendar.eternitywall.com
Got 1 attestation(s) from https://bob.btc.calendar.opentimestamps.org
```

Expected outputs after successful cache:
```
Got 1 attestation(s) from cache
```

## Protocol for transparently preparing files for consumption by other internal departments

Files that have been anonymised are ready for dissemination to other internal departments. At this point (and at any interim modification points during the process) we can use the same notarising action to confirm the state after the data has been handled by the ADC team.

On finalising a data file change, generate and submit a new commitment of the file status to the blockchain. Submit the `.ots` file alongside the data that has been shared, so that downstream departments can independently confirm the data's integrity.
```
> ots stamp <filename>
Submitting to remote calendar https://a.pool.opentimestamps.org
Submitting to remote calendar https://b.pool.opentimestamps.org
Submitting to remote calendar https://a.pool.eternitywall.com
Submitting to remote calendar https://ots.btc.catallaxy.com
> gsutil put gs://ail-blockathon-portal.appspot.com/anonymisedsources/<filename>
> gsutil put gs://ail-blockathon-portal.appspot.com/anonymisedsources/<filename>.ots
```
