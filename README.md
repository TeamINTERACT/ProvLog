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


Case Study
----------
Benoit discovered a data reversal error in the ToP files, which Kole thinks needs to be changed in ALL the datasets, even though some research has already been conducted on it. Here's a great update distribution nightmare: which files have been updated and which haven't?

In every ToP file/table, swap the 'easting' and 'northing' columns, as they were mislabeled in the output statement of Kole's ToP generation script.

NOTE: Implement the prvlog tools on cedar BEFORE correcting the archived files.


Editing and verification tool for managing PRVLOG sidecar files which are 
A sidecar editing tool for 
Datasets used to inform decision-making in the health sector
 
Analysis of these datasets has direct influence on decisions affecting the health outcomes of both individuals and entire populations. For that reason, it is imperative that the data be accurate, faithful, secure, private, and transparent, regardless of its source or any intervening transformations. In fact, those transformations need to be well documented and undoable.

Given the importance of the decisions it informs, it is entirely conceivable that the data chain will be subject to scrutiny after the fact - either academic or legal - and should be maintained in a manner that will permit such scrutiny.

In tension with that mandate, the nature of exploratory research is inherently chaotic. Processes are in a constant state of flux, data formats and structures are malleable, mistakes are made, decisions are taken, reversed, retaken, abandoned, etc. And yet, through this ever-shifting landscape, we must still somehow track what gets modified, and why, for every piece of data that contributes to the overall findings.

Two concerns to address: 

	- How to demonstrate an unbroken chain of verifications from raw data creation through to the final publication of derived datasets
	- How to document all intentional changes to the data
	- How to detect unintentional changes so they can be corrected

28431f3f8ae3397d957f04150b1eff91  hostname://path/to/file/VIC_W2_Eth_ToP_5min_20210115.zip
@ 2021-01-20 11:22:17 Original file creation. 
< 2021-01-20 11:22:17 28431f3f8ae3397d957f04150b1eff91  hostname://path/to/file/VIC_W2_Eth_ToP_5min_20210115.zip
^80e0d410f20b970439a2e8133de7781f

Structure of a .PRVLOG file:
    Top line is always: MD5SUM  FNAME
        Represents most recent checksum of the subject file
        This top line was intentionally designed to reflect the
        common format of checksum files, so that pre-existing 
        checksums can be grandfathered into the PRV chain. 

        But a runtime warning will be issued by the prv system
        if that line does not contain a machine name and ://
        prefix. This is to give the operator a chance to update that
        entry with more complete information. In practice, however, this
        information is sometimes not available, so if it is not
        remediated, the historical information will be transfered as is to the
        comment section and the new top line will contain proper machine
        and prefix syntax. (Or possibly record it with the key value UNKNOWN for 
        machine name.)
                
    Followed by any number of comment entries.

    Comment consists of an @ line and a < line.

    @ Is a timestamped comment regarding a provenance/integrity change.

    < Is a timestamped snapshot of the checksum and file path taken from 
      top line of this file prior to adding this comment.

    Last line of the file is the ^ line, which is just the checksum of all the lines of this file, prior to this line.

    Any operation on this file will fail with a warning message
    if the ^ line does not match the previous content.

So every distinct variant in the lineage of FILE is documented in
the PRV file, including its checksum value and its fully
normalized storage path and name. (Which will help us locate
backups of that file on those previous machines, if nec.)

In addition, this file also serves as its own checksum record.

PRV Scanning:
    On a PRV-enabled computer, a service runs on a regular schedule,
    searching for .prv files and verifying that the active checksum
    value for its subject file is still valid, and that the
    self-checksum at the end is also still valid.

    If the subject file checksum is ever found to be incorrect, the
    operator is notified. 

    If the mismatch is caused by an accidental alteration, the file
    can be restored and no changes are required to the PRV file. But
    if the alteration was intentional, then the operator issues
    another prv change, which updates the PRV file, along with an
    explanation of the change.

    For data stored in non-atomic form, such as tables in a DB, there
    is no simple way to run a PRV-scan. Instead, a regularly scheduled 
    pre-prv script can be run to generate a DB-fingerprint file of the 
    table, which can then be tracked within the PRV system.

    (Alternatively, the table could be intuited from the path and
recomputed at verification time, but that would require the PRV
system to have an extensible scripting system to let it probe
paths on different DBs and protocols. It's simpler to leave that
to the operator to set up in an independent cron job.)


For files that can be accommodated within git, some of these
features are redundant. After all, git can tell you when a file
has been changed. Unfortunately, git does not play nicely with
enormous datasets, cannot provide much information about the
nature of a change to binary content, and does not have access to
provenance history before it was first moved into git control on 
this machine.

prv log PRVFILE
prv log -t CHKSUMFILE PRVFILE
prv verify PRVFILE
"""
