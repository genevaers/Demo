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
  
The installation process for the GenevaERS Performance Engine will create several mainframe data sets whose names begin with your TSO Prefix.  (This is  the same as your TSO ID, unless you have changed it to something else by using the TSO PROFILE PREFIX command.)  Using the TSO Prefix as the high-level qualifier of the installation data sets allows us to avoid enclosing data set names with single-quote marks in any TSO commands mentioned below or in file transfer processes.  

1. Copy the following JCL and paste it into a JCL library member in your mainframe session: 
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
     DELETE  &HLQ..GVBDEMO.WB.XML.XMI   PURGE
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
     //WBXML    DD DSN=&HLQ..GVBDEMO.WB.XML.XMI,                             
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
1. Update the JOB statement above to conform to your installation's standards.
1. Set the value of HLQ above to your TSO Prefix. For example: 
        SET HLQ=APERSON
1. Submit the job to pre-allocate the transfer data sets.
1. In a new tab on your browser, navigate to the GenevaERS Performance Engine Demo page:  
        https://github.com/genevaers/demo
1. In the new tab from the previous step, find "Releases" on the right which is under the "About" section.  Click on the green "Latest" button.
1. Select each of these files on the Latest Release page and download them to your local drive.  
     - GVBDEMO.JCL.XMI
     - GVBDEMO.GVBLOAD.XMI
     - GVBDEMO.WB.XML.XMI
1.  Using your preferred file transfer technique, upload the following files in binary mode from your local drive to your mainframe, overwriting the transfer data sets that have just been allocated:
     - GVBDEMO.JCL.XMI
     - GVBDEMO.GVBLOAD.XMI
     - GVBDEMO.WB.XML.XMI
1.  Copy the following JCL and paste it into a JCL library member in your mainframe session:
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
       DELETE  &HLQ..GVBDEMO.WB.XML   PURGE                                  
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
        RECEIVE  INDSN(&HLQ..GVBDEMO.WB.XML.XMI)                             
                   DSN(&HLQ..GVBDEMO.WB.XML)      RELEASE                    
        RECEIVE  INDSN(&HLQ..GVBDEMO.JCL.XMI)                                
                   DSN(&HLQ..GVBDEMO.JCL)         RELEASE                    
      ```
1. Update the JOB statement above to conform to your installation's standards
1. Set the value of HLQ above to your TSO Prefix. For example:  
        SET HLQ=APERSON 
1. Submit the job to expand the transfer data sets into the installation data sets.  
1. Update the JCL in \<your-tso-prefix\>.GVBDEMO.JCL(GENDATA) according to the comments there and submit the job to generate the demo data.
1. Update the JCL in \<your-tso-prefix\>.GVBDEMO.JCL(RUNPASS1) according to the comments there and submit the job to execute the GenevaERS Demo Pass 1.  
1. Review the following control reports in your job output: 
     - MR91RPT - This report is from program GVBMR91 (the Logic Phase), which specifies the work to be done in the current run.  It also optimizes the work to allow the upcoming Extract Phase to perform multiple operations in a single pass of the source data.  For more about what the demo does, see https://github.com/genevaers/demo/blob/main/docs/WhatDemoDoes.md
     - REFRRPT - This report is from program GVBMR95R (the Reference Phase), which pre-processes reference data to conserve memory in the Extract Phase
     - EXTRRPT - This report is from program GVBMR95E (the Extract Phase), which reads one or more source data files, performs table lookups and transformations, and writes one or more output files.  
     - MR88RPT - This report is from program GVBMR88 (the Format Phase), which sorts, summarizes, and formats the data if necessary.  
1. Review the following data sets that were output from this run: 
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DAGSTATO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DCOBYSTO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DCUSTORO
     - \<your-tso-prefix>\.GVBDEMO.PASS1.DEXLKUP0

-----

## Next Topics

[What the Demo does](WhatDemoDoes.md)

[Link to Demo repo in GitHub](https://github.com/genevaers/demo)

