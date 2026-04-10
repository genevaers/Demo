---
layout: page
title: Install
nav_order: 3
---

# Install the GenevaERS Demo

Prerequisites:
- IBM z13 mainframe or later
- IBM z/OS V2R3 or later
- IBM DFSORT or compatible sort utility

## Download the release contents

The installation process for the GenevaERS Performance Engine will create several MVS datasets. It also copies a Java .jar file to a USS directory. This is the run control application (RCA). In a new tab on your browser, navigate to the GenevaERS Performance Engine Demo page.  
```
https://github.com/genevaers/demo
```

Navigate to "Releases" of the GVBDEMO repository on the right which is under the "About" section.  Click on the green "Latest" button.
Select each of these files on the Latest Release page and download them to your local drive.
```
1) GVBDEMO.JCL.XMI
2) GVBDEMO.GVBLOAD.XMI
3) GVBDEMO.WBXML.XMI
4) rcapps-4.1.0_<RCnn>.jar
```
## Pre-allocate MVS datasets

The prefix for the MVS datasets is  the same as your TSO ID.  Using the TSO ID as the high-level qualifier of the installation of the MVS datasets simplifies the installation.

Copy the following JCL and paste it into a JCL library member in your mainframe session:
```
//ALCDEMO  JOB (ACCT),CLASS=A,MSGCLASS=X,MSGLEVEL=(1,1),NOTIFY=&SYSUID.
//*
//         EXPORT SYMLIST=*
//         SET HLQ=<your-tso-prefix>
//*
//DELFILES EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *,SYMBOLS=EXECSYS
 DELETE  &HLQ..GVBDEMO.GVBLOAD.XMI  PURGE
 DELETE  &HLQ..GVBDEMO.WBXML.XMI    PURGE
 DELETE  &HLQ..GVBDEMO.JCL.XMI      PURGE
 IF MAXCC LE 8 THEN         /* IF OPERATION FAILED,     */    -
     SET MAXCC = 0          /* PROCEED AS NORMAL ANYWAY */
//*
//ALLOCATE EXEC PGM=IEFBR14
//GVBLOAD  DD DSN=&HLQ..GVBDEMO.GVBLOAD.XMI,
//            DISP=(NEW,CATLG,DELETE),
//            UNIT=SYSDA,
//            SPACE=(TRK,(3000,300)),
//            DCB=(DSORG=PS,RECFM=FB,LRECL=80,BLKSIZE=3120)             
//WBXML    DD DSN=&HLQ..GVBDEMO.WBXML.XMI,                             
//            DISP=(NEW,CATLG,DELETE),
//            UNIT=SYSDA,
//            SPACE=(TRK,(10,10)),
//            DCB=(DSORG=PS,RECFM=FB,LRECL=80,BLKSIZE=3120)
//JCL      DD DSN=&HLQ..GVBDEMO.JCL.XMI,
//            DISP=(NEW,CATLG,DELETE),
//            UNIT=SYSDA,
//            SPACE=(TRK,(10,10)),
//            DCB=(DSORG=PS,RECFM=FB,LRECL=80,BLKSIZE=3120)
```
Update the JOB statement above to conform to your installation's standards. Set the value of HLQ above to your TSO Prefix. Submit the job to pre-allocate the transfer data sets.

## Upload XMI files corresponding to the MVS datasets to your mainframe

Using your preferred file transfer technique, upload the following files in binary mode from your local drive to your mainframe, overwriting the transfer data sets that have just been allocated:

1) GVBDEMO.JCL.XMI
2) GVBDEMO.GVBLOAD.XMI
3) GVBDEMO.WBXML.XMI

## RECEIVE XMI files to extract MVS datasets

Copy the following JCL and paste it into a JCL library member in your mainframe session:
```
//RCVDEMO  JOB (ACCT),CLASS=A,MSGCLASS=X,MSGLEVEL=(1,1),NOTIFY=&SYSUID.
//*                                                                    
//         EXPORT SYMLIST=*                                            
//         SET HLQ=<your-tso-prefix>                                             
//*                                                                    
//DELFILES EXEC PGM=IDCAMS                                             
//SYSPRINT DD SYSOUT=*                                                 
//SYSIN    DD *,SYMBOLS=EXECSYS                                        
 DELETE  &HLQ..GVBDEMO.GVBLOAD  PURGE                                  
 DELETE  &HLQ..GVBDEMO.WBXML    PURGE                                  
 DELETE  &HLQ..GVBDEMO.JCL      PURGE                                  
 IF MAXCC LE 8 THEN         /* IF OPERATION FAILED,     */    -        
     SET MAXCC = 0          /* PROCEED AS NORMAL ANYWAY */             
//*                                                                    
//RECEIVE  EXEC PGM=IKJEFT01,DYNAMNBR=30                               
//SYSTSPRT DD SYSOUT=*                                                 
//SYSTSIN  DD *,SYMBOLS=EXECSYS                                        
  PROFILE NOPREFIX                                                     
  RECEIVE  INDSN(&HLQ..GVBDEMO.GVBLOAD.XMI)                            
             DSN(&HLQ..GVBDEMO.GVBLOAD)     RELEASE                    
  RECEIVE  INDSN(&HLQ..GVBDEMO.WBXML.XMI)                             
             DSN(&HLQ..GVBDEMO.WBXML)       RELEASE                    
  RECEIVE  INDSN(&HLQ..GVBDEMO.JCL.XMI)                                
             DSN(&HLQ..GVBDEMO.JCL)         RELEASE                    
```
Update the JOB statement above to conform to your installation's standards. Set the value of HLQ above to your TSO Prefix. Submit the job to expand the transfer data sets into the installation data sets.

## Allocation USS directory to contain RCA .jar file

Logging onto USS use mkdir to create a directory under your user ID
```
/u/<userid>/git/public/RCA_jar
```
## Copy RCA .jar file to USS directory

Using your preferred file transfer technique, upload the following file in binary mode from your local drive to the zFS file system of your mainframe:
```
rcapps-4.1.0_<RCnn>.jar
```
## Creating symbolic link and adding permissions
```
touch rcapps-latest.jar;
rm rcapps-latest.jar;
ln -s rcapps-4.1.0_<RCnn>.jar rcapps-latest.jar
```
# Generate the DEMO data

Update the JCL in <your-tso-prefix>.GVBDEMO.JCL(GENDATA) according to the comments there and submit the job to generate the demo data.

# GVBDEMO JCL

The DEMO comprises the following jobs:

1) RUNEXT1
2) RUNFMT1
3) RUNFMT2
4) RUNFMT3

Update the JCL in <your-tso-prefix>.GVBDEMO.JCL library according to the comments there and submit the first job RUNEXT1 to execute the GenevaERS Demo Run Control and Extract phases. This job will submit the 3 Format phase jobs upon completion.

## Review the following control reports in your job output

  RCARPT - This report is from program RCA (the Run Control Phase), which specifies the work to be done in the current run.  It also optimizes the work to allow the upcoming Extract Phase to perform multiple operations in a single pass of the source data.

  REFRRPT - This report is from program GVBMR95R (the Reference Phase), which pre-processes reference data to conserve memory in the Extract Phase

  EXTRRPT - This report is from program GVBMR95E (the Extract Phase), which reads one or more source data files, performs table lookups and transformations, and writes one or more output files.

  MR88RPT - This report is from program GVBMR88 (the Format Phase), which sorts, summarizes, and formats the data if necessary.  

## Review the following data sets that were output from this run: 

     - \<your-tso-prefix>\.GVBDEMO.PASS1.DAGSTATO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DCOBYSTO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DCUSTORO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DEXLKUP0

## Next Topics

[What the Demo does](WhatDemoDoes.md)

[GenevaERS Documentation](https://genevaers.github.io/User-Documentation/)

[The GenevaERS Project](https://genevaers.org/)~~~~

[Link to Demo repo in GitHub](https://github.com/genevaers/demo)

