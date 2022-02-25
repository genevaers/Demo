---
layout: page
title: After the Demo
nav_order: 89
---
# After you have run the Demo

## Scale up data and see GenevaERS's Reaction

The Data Generator program allows you to increase the amount of demo data created on your system.  You can increase the data volumes from the default values, and see how the GenevaERS Performance Engine reacts.

1. In the JCL Member DTGENRUN, change the following parameter to increase or decrease the number of orders contained in the demo file.  The default is 5,000,000 orders. 
```
//         SET ORDERMAX=5000000
```
2. If generating more than 2 billion orders, adjust the space allocation parameters on the ORDER and each of the Order Item (ORDITMnn) output datasets from the GVBDTGEN program.  
   
   1. Each order (the parameter above) generates about 47 bytes. 
   ``` 
        //ORDER    ...
        //            SPACE=(TRK,(5000,500),RLSE),
     ```
   2. Order Items are 5.5 times the Orders, and are 36 bytes long.
   ```
        //ORDITM01 ...
        //            SPACE=(TRK,(9000,500),RLSE),
        //ORDITM02 ...
        //            SPACE=(TRK,(9000,500),RLSE),
        //ORDITM03 ...
        //            SPACE=(TRK,(9000,500),RLSE),
   ```

3. Run the DTGENRUN job to generate the new data volumes.
   
4. Run the RUNEXT1 again.  
   
5. Inspect the EXTRPT report in the RUNEXT1 sysout to see the impact on the run of the differing data volumes.  

## Do your own processing in GenevaERS

When you have run the Demo and seen what GenevaERS can do, the next step is configure your own processing.

The way to do that is in a part of GenevaERS called the Workbench.  You will be able to see how the Demo was configured, and then configure your own processing.

To download and run the Workbench, see [GenevaERS workbench doc](https://genevaers.github.io/wb/).
