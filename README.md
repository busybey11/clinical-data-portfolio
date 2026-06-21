# Clinical Trial Data Quality Control Pipeline

A quality control pipeline built using Python that replicates the kind of data validation checks a Clinical Data Manager or Clinical Data Coordinator performs on incoming clinical trial data, built using real, publicly released clinical trial data.

## Overview

Clinical trial data does not arrive clean. Before it can be locked, analyzed, or submitted to a regulator, every domain has to pass a series of validation checks, including missing fields, implausible values, duplicate records, and illogical dates. This project replicates four of the most common checks a Clinical Data Manager runs, applied to a real, anonymized clinical trial dataset. The goal was not just to flag problems, but to interpret them the way a real reviewer would, distinguishing genuine data issues from expected clinical patterns and from data that is incomplete but still valid.

## Data Source

This project uses the CDISC Pilot Project Data, a real clinical trial dataset with patient identities removed, which CDISC, the Clinical Data Interchange Standards Consortium, distributes publicly for training purposes. The source can be found at the cdisc org slash sdtm adam pilot project repository on GitHub.

The project draws on three domains. The demographics domain is known as DM. The vital signs domain is known as VS. The adverse events domain is known as AE.

## Checks Performed

Four checks were performed. The first check looks for missing required fields in the demographics domain, specifically the subject identifier, age, and sex. The second check flags vital sign values in the VS domain, such as systolic blood pressure, diastolic blood pressure, and pulse, that fall outside plausible physiological ranges. The third check looks for duplicate subject identifiers in the demographics domain. The fourth check compares adverse event start dates against each subject's reference start date in the demographics domain, in order to flag adverse events that appear to have started before the trial began for that subject.

## Results

The pipeline flagged seventy two issues in total. Zero subjects were missing required fields. Zero duplicate subject identifiers were found. Seven vital sign readings fell outside the expected range. Forty five adverse event dates were confirmed to fall before the subject's reference start date. Twenty adverse event dates appeared to fall before the reference start date, but only at reduced precision, meaning they need manual review rather than being treated as confirmed issues.

## Key Insight: Why a Flat Issue Count Is Misleading

The adverse event date check initially flagged sixty five records as having an adverse event that started before the study reference date, which sounds alarming until it is examined more closely. Two very different situations were hidden inside that single number.

The first group, forty five records, had complete dates written in full year, month, and day, with a gap of only a matter of days to a couple of months before the reference start date. These are almost certainly adverse events collected during the screening period, meaning the time between when a subject gives informed consent and when they receive their first dose. Capturing adverse events during this window is standard clinical trial design, not a data error.

The second group, twenty records, had only partial dates, recorded as just a year or as a year and month, and in some cases the gap was decades long. For example, one adverse event was dated only as nineteen eighty six against a reference start date in two thousand thirteen. These records are almost certainly previously existing conditions that were logged with only an approximate year of original onset, not adverse events from the screening period at all.

Treating both groups as the same kind of issue would have hidden this real distinction. The refined check separates the two groups by how precise each date is, and calculates the actual number of days between dates whenever full precision is available, so the final output tells a clinically meaningful story instead of giving a single flat count.

## Limitations and Notes

There are a few limitations worth noting. The vital sign ranges used in this project are illustrative rather than official clinical reference ranges. In an actual study, these thresholds would come from the protocol's data validation plan, and they may vary depending on the patient population, for example pediatric studies typically use different thresholds than studies involving older adults.

Flagged values should be treated as candidates for a data query rather than confirmed errors. In real clinical data management practice, an out of range or illogical value generates a query back to the study site for confirmation, and it is not assumed to be wrong or automatically corrected.

Finally, SAS transport files, which carry the xpt extension, load character columns as raw bytes when read with the pandas read sas function. This pipeline includes an explicit step to decode those columns into readable text, which avoids silent comparison problems and unreadable output.

## Tools Used

This project was built using Python, with the pandas library for data handling and the pyreadstat library for reading SAS transport files, all run inside Google Colab.

## How to Run

To run this project, open the notebook in Google Colab and run all of the cells in order. The notebook pulls the CDISC Pilot data directly from GitHub, so no manual download is required. The final cell produces a categorized summary and saves the complete report to a file named clinical data qc report dot csv.

## Possible Future Enhancements

There are a few ways this project could be extended in the future. It could be expanded to cover additional domains, such as concomitant medications or medical history. It could also be enhanced to automatically draft query text for each flagged record, similar to a real clinical data management query log. Finally, the same precision aware date logic used for adverse event dates could be applied to other date comparisons across different domains.

This project was built by Beah V. Vega as part of a clinical data management portfolio focused on contract research organizations.
