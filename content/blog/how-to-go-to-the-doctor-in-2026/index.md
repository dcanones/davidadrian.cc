---
title: "How to go to the doctor in 2026"
date: 2026-08-30T12:00:00.000Z
draft: false
description: "A practical guide to preparing for medical consultations, owning your medical records, and advocating for yourself in a saturated healthcare system."
tags:
  - "Healthcare"
  - "Personal"
featureimage: "feature.jpg"
---

> **Warning:** this is not AI slop. This is advice that could potentially save your
> life (or at least spare you serious trouble, and money) in the future.

## Why this article

I have been ruminating about writing this for a long time. It comes from my experience
of my wife's serious illness in her early thirties. What could we have done
differently that would have improved the outcome for us? What did we do, after her
diagnosis, that helped?

This article will be especially helpful for young adults: people in their twenties or
early thirties whose health has been outsourced to their parents for most of their
lives, and who very probably have never experienced a serious health issue (yet).

Nobody teaches you how to properly go to the doctor, and when you really need that
skill (and believe me, you will) it's too late to fast-track it. So you'd better read
this carefully, and apply it.

## Part 1: Don't blindly trust the system. You're the only one in charge of your health.

One of the most common traps many people fall into is blindly trusting the health
system, which can extend to specific hospitals or doctors.

Healthcare has changed substantially over the last 20–30 years. My uncle, a retired
rheumatologist with 40 years in the Spanish public healthcare system, answered like
this when I asked him about the most fundamental change he witnessed over his career:
"At the end of my career in 2025, we had much more advanced technology for diagnosis
and treatment options for patients, but the healthcare system is overall much more
saturated, and human capital much more scarce."

The consequence of this change is that hospitals and clinics operate these days like
factories, lacking the personalization and personal touch that healthcare used to
have.

When you visit a doctor, there is probably a time cap for the consultation (something
like 10 minutes), plus a lot of administrative work they also have to finish on time.
You are just one more number in a long list the doctor has to clear before finishing
their shift.

Given this new situation in healthcare, it is clear that you can no longer just
delegate your health to the system and trust that everything is going to be OK without
your close supervision and active intervention. If you just "let the system work,"
there is a high chance you are not going to be properly diagnosed, and thus treated.

## Part 2: Before a consultation, always prepare properly

A doctor's time is probably the scarcest resource in healthcare these days, so you must
make the most of it to achieve the best possible outcome. This means preparing for the
consultation so that:

- The doctor can understand what is going on with your health in one minute, and
  doesn't spend the whole consultation trying to collect your clinical history.
- The doctor can invest their precious time in what is really needed: addressing all
  the information, filling the gaps if needed, diagnosing, and deciding on your
  treatment.

Very important: hospital and clinic software (used to store your medical history) is
old, probably very slow, and totally fragmented (it changes from hospital to
hospital). Do not assume the doctor has access to your clinical history or reports. If
they do, the terrible UX of these tools makes it difficult for them to find the
relevant information, or even to access it in time.

My recommendation is the following: always do your homework before a consultation.

1. Collect the relevant medical reports related to your specific health problem. Use
   common sense: if you're visiting the orthopaedic surgeon because of a broken leg,
   you don't need to include a report about the flu.
   Include: medical reports, medical imaging reports if it makes sense, and your most
   recent blood tests if you have them. Print them on paper, and also save them as PDF
   and store them on a USB drive that you can give to your doctor.
2. Collect all your medical imaging tests (CT scans, ultrasound, and MRI) and place
   them (unzipped) together with the radiologist's report on the previously mentioned
   USB drive, plus burn them to a DVD (yes, remember that healthcare software and
   hardware can be old, and some hospitals and clinics have USB drive limitations).
3. Prepare a one-page summary of your case. Focus on storytelling, and use a timeline
   as the backbone of the document. Something like this:
   - *January 2019:* Broke my ankle while playing soccer. First treatment at XXX
     hospital. CT scan (`2019_01_12__ct_report_ankle__xxx_hospital.pdf`) shows...
   - *February 2019:* Had surgery at XXX hospital. Results are... More information in
     `2019_02_17__surgery_report__xxx_hospital.pdf`.
   - *Etc.*

   It is very important that this document is as short as possible, very easy to
   understand, and self-explanatory. This is probably the only document the doctor is
   going to read during the consultation.
4. At the end of the document, include the goal of the consultation and your
   questions.

Do this for every first-time consultation, and for follow-up consultations in
hospitals and clinics where it is not guaranteed that the same doctor will follow up
on your case. Doctors, even in the same hospital and specialty, do not always talk to
each other. Preparing all this is tedious, but after the first time, updating and
keeping it current is much easier, and with the use of AI tools, there is no excuse.

## Part 3: Always ask for your medical reports

As you already know, the cornerstone of this methodology is information. Keeping it
current and organized is vital.

Today, doctors spend more time performing bureaucratic tasks (like writing reports)
than examining their patients. At least try to get something out of it.

After a consultation (new or follow-up), a medical imaging test, or even a simple
blood test, always download and store:

- For medical imaging tests, the RAW file (can be several GB). Download it from the
  patient portal of your hospital or clinic if possible. If not, ask for it in
  person: you have the right to own your medical data. It will be helpful in case you
  have to switch to a new hospital or clinic, or need a second opinion elsewhere.
- For consultations (new or follow-up), medical imaging tests, blood tests, etc., ask
  for the PDF report. You can probably download it from your patient portal, but ask
  for it in person if that's not possible.

This information must be stored both on a physical drive (HDD) and in the cloud
(Dropbox, Google Drive, etc.) for redundancy, and properly organized. This is the
ontology I use:

```text
health/
└── <person>/
    ├── reports/                   # text reports, PDF
    │   ├── consultation/
    │   │   ├── emergency/
    │   │   ├── discharge/
    │   │   ├── follow_up/
    │   │   └── second_opinion/
    │   ├── pathology/
    │   ├── blood_tests/
    │   ├── imaging/               # radiologist reports
    │   │   └── 2019_01_12__ct_report_ankle__xxx_hospital.pdf
    │   ├── other/
    │   └── _old/                  # before the current episode
    └── images/                    # imaging studies, raw
        └── 2019_01_ankle_ct/
            ├── 2019_01_12__ct_report_ankle__xxx_hospital.pdf
            ├── info.json
            └── SER_0001 … SER_0012     # raw DICOM series
```

File naming: `YYYY_MM_DD__document_type__center.pdf`. The folder names above are in
English for this article. Name yours in whatever language you're comfortable with.
What matters is the shape of the structure, not the words.

## Part 4: Don't settle

Don't let your shyness, introversion, or blind trust in the system jeopardize your
health. This is one of the main mistakes my wife and I made.

- **During the consultation:** if you think your doctor is missing something important
  or underestimating something, say so! If you're not sure whether the doctor has seen
  the medical reports or imaging tests you sent by email prior to the consultation
  (remember to always carry them too), ask! If you have questions and/or are worried
  about something, ask! Doctors are human and, especially nowadays, overworked. They
  forget things and make mistakes, so it is your responsibility to support and oversee
  them. This is extremely important if no single doctor is following your case
  (having one is something you should aim for if possible). If you're not being
  followed by one doctor but by a group of doctors, don't assume they are properly
  coordinated. They probably aren't. You will have to be very careful and make the
  effort to keep them in sync.
- **After the consultation:** there is a chance that your future appointments or
  diagnostic tests get delayed for whatever reason. Remember that hospitals and
  clinics are operating more and more like factories, and they pay attention to
  certain KPIs. One of them is the number of open issues and complaints. It's sad,
  and even a bit controversial, but if you really need things to speed up, go in
  person and complain. Give good reasons while doing it (e.g., that you need the test
  result before your next appointment), but do it. I remember an important medical
  imaging test (a PET-CT scan) for my wife that went overdue. We went to the hospital
  to find out what was going on, and to stress the vital importance of that test. They
  told us that one of the two PET-CT scanners was under repair and there was a delay.
  I remember that ten minutes after we left, my wife got a phone call to schedule the
  appointment. Luck? I don't think so.

  If you have to spend a morning waiting in a queue at a hospital to get your
  appointments and tests prioritized, it is worth it 100%.
- If possible, coordinate with your doctor to speed things up. For example, in Spain
  (where I live), the healthcare system is public and tax-funded. It's good in terms
  of coverage and treatments available, but also slow, bureaucratic, and crowded. Many
  people combine this system with private insurance. Many doctors who work in public
  hospitals on an 8-to-3 schedule also work in private clinics in the evening. If you
  have access to that, you can also speed up tests (like medical imaging) or even,
  sometimes, surgeries, as queues are normally shorter in private healthcare (again,
  this is location-specific).

## Part 5: Leverage AI

Most of the advice in this article requires an investment of considerable time and
effort. A very interesting tool that can help you save a lot of that time is AI.
Here's how I use it:

- **Organize the raw files.** Remember the ontology example from Part 3? It requires
  processing each new medical record, renaming the file, and placing it in the correct
  folder so you can find it easily later. Using a tool like Claude Cowork, you can
  just point it to your medical record files and ask it to organize them following
  that ontology. This is especially helpful the first time you're building it.
- **Centralize and normalize the information.** I keep a text-based, centralized
  repository with all my (and my wife's) medical information, processed and
  normalized. For that, I use a section of my Obsidian vault, but it can be virtually
  any text document (Google Docs, MS Word, etc.). The real value of AI here is that
  for each medical report you have, you can ask an AI to process it, normalize it, and
  output it as text integrated into your central repository. This makes it extremely
  easy to generate a summary for your doctor before a consultation, or to analyze your
  data with more context. For example, I keep a table with all my blood test
  measurements from the last few years, and use it to find trends and relate results
  to nutrition, sports, sleep, and illnesses.

## Conclusion

If you take only one thing from this article, remember this: **you are the one in
charge of your health. Do not outsource it. Do not blindly trust the healthcare system
to take care of you.**

A disclaimer: this article is probably biased toward the particularities of Spain,
where I live. But I'm sure that much of this advice generalizes well to other
locations and systems. Use your common sense, and apply what is useful for you.
