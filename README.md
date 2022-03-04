Provenance Logger
=================

The Problem
------------

Health Research presents an almost perfect storm of data management headaches. Some files are small and created/entered by hand (doctors notes, patient diaries, transcribed interviews, etc.) while others are enormous and generated automatically (GPS tracking, accelerometry, biometrics, etc.). These datasets are typically contributed by a large number of different users, each with widely varying degrees of experience and proficiency working with data, while working on different computers, each with potentially different operating systems and software tools. 

The data collected often includes not just directly personal information, but also secondary information that could be used to identify the contributors through indirect means. Clearly, such datasets require a high degree of privacy protection and security, but at the same time, the results gleaned from these datasets are of paramount societal importance, contributing to decisions that will inform both individual health outcomes and nation-wide economic policies, so it is imperative that the data itself, the methodologies, and the records of its creation be able to withstand academic, commercial, political, and possibly even legal scrutinies.

In support of this mandate, there are 5 domains of data integrity that we have seen discussed in the procedures and methodologies literature: network security, physical security, data encryption, data redundancy (backups), and revision control. But in scanning these contributions to help facilitate our own practice, there was a 6th domain of concern to us that was not sufficiently addressed: data provenance.

It's true that the comment logs in an Revision Control System (RCS) repository (such as git or svn) can be used as a rudimentary sort of "provenance log," but there are a number of ways in which this is insufficient. First, consider that some data files are simply too large to be effectively tracked within most RCS tools, so they would have no such record, even though they often comprise the primary data. Second, many of the files that contribute to a dataset go through some kind of migratory life-cycle BEFORE they reach the collection system, so there is a gap in the provenance record at its very beginning, which is often the most volatile portion of the life-cycle, as it is managed by less expert handlers. Ideally, a provenance system should be able to track and verify the integrity of the data at every step from the time of its creation onward.

Some literature has been written about the need to track data alterations, but most of it has been focussed on the need to track changes made by researchers who are using a published dataset. To date, we have found nothing about tracking provenance PRIOR to publication - tracking the history of the data elements that were used to create the dataset in the first place.


Our Solution
------------
This is where Provenance Logger (hereafter referred to as ProvLog) comes in. It is by no means offered as a complete solution, but is instead intended as a stop-gap, a bare minimum, to use until a more robust infrastructure emerges in the field.

To that end, ProvLog was designed to be extremely flexible, OS agnostic, lightweight, and open source. This initial implementation is a command-line-only tool, but it can be run on any computer that supports Python.

What can be monitored
    file content
    directory listings
    SQL table content

What is logged
    date and time of log entry
    target URI
    current MD5 checksum
    log message

How it is tracked
    record checksum when target is added to tracking system
    regular (frequency determined by operator) verify checksums
    if checksums have changed, alert operator by email or log file
    operator then triggers an update which adds a new checksum to the log, along with an explanation of the change

Where it is tracked
    .prvlog sidecar files

Caveats:
    ProvLog relies on existing OS mechanisms to manage file access privileges and is intentded to guard against ACCIDENTAL data corruption. If MALICIOUS corruption is a concern, consider deploying one of the blockchain-based tools instead.

