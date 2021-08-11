# How To Install the GenevaERS Performance Engine Demo

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
2. Update the JOB statement above to conform to your installation's standards.
3. Set the value of HLQ above to your TSO Prefix. For example:
        SET HLQ=APERSON
4. Submit the job to pre-allocate the transfer data sets.
5. Navigate to the GenevaERS Performance Engine Demo page (if you're not there already): https://github.com/genevaers/demo
6. Press the green "Latest" button in the Releases section at the right of this page.   
7. Select each of these files on the Latest Release page and download them to your local drive.  
     - GVBDEMO.JCL.XMI
     - GVBDEMO.GVBLOAD.XMI
     - GVBDEMO.WB.XML.XMI
8.  Using your preferred file transfer technique, upload the following files in binary mode from your local drive to your mainframe, overwriting the transfer data sets that have just been allocated:
     - GVBDEMO.JCL.XMI
     - GVBDEMO.GVBLOAD.XMI
     - GVBDEMO.WB.XML.XMI
9.  Copy the following JCL and paste it into a JCL library member in your mainframe session:
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
12. Update the JOB statement above to conform to your installation's standards
13. Set the value of HLQ above to your TSO Prefix. For example:
        SET HLQ=APERSON 
14. Submit the job to expand the transfer data sets into the installation data sets.  
15. Update the JCL in \<your-tso-prefix\>.GVBDEMO.JCL(GENDATA) according to the comments there and submit the job to generate the demo data.
16. Update the JCL in \<your-tso-prefix\>.GVBDEMO.JCL(RUNPASS1) according to the comments there and submit the job to execute the GenevaERS Demo Pass 1.  
17. Review the following control reports in your job output: 
     - MR91RPT
     - REFRRPT
     - EXTRRPT 
     - MR88RPT
18. Review the following data sets that were output from this run: 
     - GVBDEMO.PASS1.DAGSTATO
     - GVBDEMO.PASS1.DCOBYSTO
     - GVBDEMO.PASS1.DCUSTORO
     - GVBDEMO.PASS1.DEXLKUP0