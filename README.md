# CompetitiveCLS_ModelFit
In this project we upload the scripts needed to fit competitive aging as discussed in Avelar-Rivas et al 2019
  
  
  
  
## Calculate G and S using linear model fitting

Here we describe how to run scripts that fit plate measurements of aging fluorescent co-cultures to the model Ax+ST+Gt+CTt = ln(RFP/CFP) .

Please find functions in [Github_AbrahamAvelar](https://github.com/AbrahamAvelar)  
Most of them are in this [repository](https://github.com/AbrahamAvelar/Comparacion_Metodos_Envejecimiento/tree/master/Functions/CorrerModeloNS_ScriptsEGG)  
[Here](https://abrahamavelar.github.io/LinearModelCLS/Example) You can download sample Data and run scripts with a real example.

  
### The goal is to run the function ModelASGC as follows:  
```
[bgdataAll_LM,data2_LM]=ModelASGC(bgdata,plt,refs,OnlyMutStrain,OnlyRefStrain,medicionesminimas,exp,extraPlRefs);

```

In order to get there, we select only those measurements in exponential phase using **ExtractExponentialPoints** after preparing input variables.

```markdown
# Prepare input variables
refs=[i k]; %Integers with the number of the well(s) in which there are competitions WTrfp+WTcfp  
plt=[l m]; %Integers of the plates for which you want the model to run.  
Tiempo0=1; % 1, so it includes the first measurement of each outgrowth before the exponential time points.  
showfig=1; % 1 if you wish to see growth curves with the exponential phase time points highligthed  
-  %BgDataAll is the structure with fields OD, RFP, CFP and t that comes from either LoadTecanFiles or from bgdataAll2BgDataAll  
    
# Execute exponential phase point extractor script
[ExpBgDataAll = ExtractExponentialPoints(BgDataAll, plt, showfig, refs, Tiempo0 )]
```
  
This is specially useful when you have many measurements per day (more than 3) and it will also make the aproximation of G more dependent of the phase in which cells are growing exponentially.
  
    
The next step is to order the variables of time in the structure ExpBgDataAll using **calculaTiempos**
  
```markdown
# Prepare input variables
odTh=0.22; %Threshold of the change  between consecutive measurements to be identified as a new outgrowths's measurement.
  
# Execute time ordering script
ExpBgDataAll = calculaTiempos(ExpBgDataAll, plt, odTh);
```
It may be a good idea to play a bit with odTh depending of the strains, media and other factors that may change the dynamics or the magnitude of the growth curve with OD600. Usually values between 0.2 and 0.4 work just fine to detect at which measurements there is a new outgrowth. 

Now we subtract background fluorescences. We prepare variables and use **restarFondoFPs**.

```markdown
# Prepare input variables
exp=1; % We put 1 because we have a structure only with exponential points which is the output of 'ExtractExponentialPoints'
bgdataPrueba=BgDataAll2bgdataEGG(ExpBgDataAll,plt,'CFP','RFP',exp); 
OnlyMutStrain = [i]; %Index or Indexes of the reference wells that have only WTcfp
OnlyRefStrain = [j]; %Index or Indexes of the reference wells that have only WTrfp (the same FP as all of the mutants)
  
# Execute background subtraction script
BgDataSinFondo = restarFondoFPs(bgdataPrueba, plt, OnlyMutStrain,OnlyRefStrain)
```

Finally we are ready to run our fitting function. It will build the predictor variables matrix for each plate and then fit them to their respective response vector using Matlab's regress function.

```markdown
# Prepare input variables
medicionesminimas=[i]; %Smallest number of valid measurements per well to be included in the fitting function
extraPlRefs = 0; % The number of an extra plate that has references in it. Put 0 if you have references in every plate.
# Execute Linear Model fitting script
[bgdataAll_LM,data2_LM]=ModelASGC(BgDataSinFondo,plt,refs,OnlyMutStrain,OnlyRefStrain,medicionesminimas,exp,extraPlRefs);
```

If you got here and the output bgdataAll_LM that contains your dataset and the vectors of solutions A, S, G and C it means you've made it to work! congratulations!


Please provide us any feedback you may have!

