Background
----------

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement – a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways. More information is
available from the website here:
<a href="http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har" class="uri">http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har</a>
(see the section on the Weight Lifting Exercise Dataset).

Synopsis
--------

The goal of your project is to predict the manner in which they did the
exercise. This is the “classe” variable in the training set.

Analysis
--------

Loading useful libraries:

    # load library
    library(caret)

    ## Loading required package: lattice

    ## Loading required package: ggplot2

First we download the data and load it into training and testing:

    # Download the training and testing datasets
    if (!file.exists("pml-training.csv")) {
      download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv","pml-training.csv",method = "curl")
    }
    if (!file.exists("pml-testing.csv")) {
      download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv","pml-testing.csv",method = "curl")
    }
    training = read.csv("pml-training.csv", na.strings=c('#DIV/0!', '', 'NA'), header=T)
    testing = read.csv("pml-testing.csv", na.strings=c('#DIV/0!', '', 'NA'), header=T)

There are 159 variables and one outcome:

    length(names(training))

    ## [1] 160

We split the training set into two datasets; validation (30%) and
training set (70%):s

    inTrain = createDataPartition(training$classe,
    p=0.70, list=FALSE)
    validation = training[-inTrain,]
    training = training[inTrain,]

We now proceed to dimension reduction by removing near zero variables
and variables with mostly NAs:

    # Remove near zero variables
    nzv <- nearZeroVar(training)
    training <- training[,-nzv]
    validation <- validation[,-nzv]

    # Remove mostly NA variables
    mostlyNA <- sapply(training,function(x) mean(is.na(x))) > 0.95
    training <- training[,mostlyNA==FALSE]
    validation <- validation[,mostlyNA==FALSE]

    length(names(training))

    ## [1] 59

We went from 159 variables to 58 variables.

    (unique(training$classe))

    ## [1] "A" "B" "C" "D" "E"

Model training:

    modelFit = train(classe ~ .,
                     method="gbm",
                     trControl = trainControl(method="repeatedcv",number = 5,repeats = 1),
                     data=training)

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.4517
    ##      2        1.3259             nan     0.1000    0.3101
    ##      3        1.1328             nan     0.1000    0.2411
    ##      4        0.9840             nan     0.1000    0.1978
    ##      5        0.8619             nan     0.1000    0.1584
    ##      6        0.7627             nan     0.1000    0.1443
    ##      7        0.6747             nan     0.1000    0.1224
    ##      8        0.5997             nan     0.1000    0.1075
    ##      9        0.5339             nan     0.1000    0.0888
    ##     10        0.4788             nan     0.1000    0.0848
    ##     20        0.1652             nan     0.1000    0.0254
    ##     40        0.0236             nan     0.1000    0.0034
    ##     60        0.0041             nan     0.1000    0.0004
    ##     80        0.0010             nan     0.1000    0.0001
    ##    100        0.0003             nan     0.1000    0.0000
    ##    120        0.0001             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7760
    ##      2        1.1448             nan     0.1000    0.4620
    ##      3        0.8700             nan     0.1000    0.3211
    ##      4        0.6791             nan     0.1000    0.2375
    ##      5        0.5379             nan     0.1000    0.1813
    ##      6        0.4298             nan     0.1000    0.1412
    ##      7        0.3455             nan     0.1000    0.1114
    ##      8        0.2790             nan     0.1000    0.0886
    ##      9        0.2260             nan     0.1000    0.0711
    ##     10        0.1835             nan     0.1000    0.0572
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0006             nan     0.1000    0.0002
    ##     60        0.0001             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000    0.0000
    ##    100        0.0000             nan     0.1000    0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7760
    ##      2        1.1448             nan     0.1000    0.4623
    ##      3        0.8701             nan     0.1000    0.3211
    ##      4        0.6792             nan     0.1000    0.2370
    ##      5        0.5379             nan     0.1000    0.1815
    ##      6        0.4298             nan     0.1000    0.1415
    ##      7        0.3455             nan     0.1000    0.1116
    ##      8        0.2789             nan     0.1000    0.0886
    ##      9        0.2260             nan     0.1000    0.0711
    ##     10        0.1834             nan     0.1000    0.0572
    ##     20        0.0242             nan     0.1000    0.0073
    ##     40        0.0006             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000   -0.0000
    ##     80        0.0000             nan     0.1000    0.0000
    ##    100        0.0000             nan     0.1000    0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.4510
    ##      2        1.3256             nan     0.1000    0.3094
    ##      3        1.1324             nan     0.1000    0.2400
    ##      4        0.9842             nan     0.1000    0.1976
    ##      5        0.8624             nan     0.1000    0.1585
    ##      6        0.7633             nan     0.1000    0.1446
    ##      7        0.6751             nan     0.1000    0.1226
    ##      8        0.6001             nan     0.1000    0.1077
    ##      9        0.5343             nan     0.1000    0.0890
    ##     10        0.4790             nan     0.1000    0.0851
    ##     20        0.1655             nan     0.1000    0.0278
    ##     40        0.0235             nan     0.1000    0.0032
    ##     60        0.0039             nan     0.1000    0.0006
    ##     80        0.0008             nan     0.1000    0.0001
    ##    100        0.0002             nan     0.1000    0.0000
    ##    120        0.0001             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7744
    ##      2        1.1446             nan     0.1000    0.4629
    ##      3        0.8698             nan     0.1000    0.3213
    ##      4        0.6789             nan     0.1000    0.2374
    ##      5        0.5376             nan     0.1000    0.1810
    ##      6        0.4296             nan     0.1000    0.1415
    ##      7        0.3453             nan     0.1000    0.1115
    ##      8        0.2788             nan     0.1000    0.0888
    ##      9        0.2258             nan     0.1000    0.0712
    ##     10        0.1833             nan     0.1000    0.0572
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000    0.0000
    ##    100        0.0000             nan     0.1000    0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7772
    ##      2        1.1447             nan     0.1000    0.4620
    ##      3        0.8700             nan     0.1000    0.3208
    ##      4        0.6791             nan     0.1000    0.2374
    ##      5        0.5377             nan     0.1000    0.1815
    ##      6        0.4296             nan     0.1000    0.1413
    ##      7        0.3454             nan     0.1000    0.1114
    ##      8        0.2789             nan     0.1000    0.0888
    ##      9        0.2258             nan     0.1000    0.0710
    ##     10        0.1834             nan     0.1000    0.0573
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.4482
    ##      2        1.3262             nan     0.1000    0.3078
    ##      3        1.1337             nan     0.1000    0.2419
    ##      4        0.9847             nan     0.1000    0.1980
    ##      5        0.8627             nan     0.1000    0.1582
    ##      6        0.7636             nan     0.1000    0.1445
    ##      7        0.6749             nan     0.1000    0.1222
    ##      8        0.6000             nan     0.1000    0.1079
    ##      9        0.5340             nan     0.1000    0.0891
    ##     10        0.4790             nan     0.1000    0.0847
    ##     20        0.1659             nan     0.1000    0.0280
    ##     40        0.0236             nan     0.1000    0.0035
    ##     60        0.0041             nan     0.1000    0.0004
    ##     80        0.0011             nan     0.1000    0.0001
    ##    100        0.0003             nan     0.1000    0.0000
    ##    120        0.0001             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7763
    ##      2        1.1447             nan     0.1000    0.4616
    ##      3        0.8699             nan     0.1000    0.3210
    ##      4        0.6790             nan     0.1000    0.2373
    ##      5        0.5377             nan     0.1000    0.1810
    ##      6        0.4297             nan     0.1000    0.1409
    ##      7        0.3455             nan     0.1000    0.1115
    ##      8        0.2790             nan     0.1000    0.0887
    ##      9        0.2259             nan     0.1000    0.0711
    ##     10        0.1834             nan     0.1000    0.0572
    ##     20        0.0242             nan     0.1000    0.0072
    ##     40        0.0008             nan     0.1000    0.0001
    ##     60        0.0001             nan     0.1000   -0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7757
    ##      2        1.1446             nan     0.1000    0.4618
    ##      3        0.8699             nan     0.1000    0.3206
    ##      4        0.6790             nan     0.1000    0.2372
    ##      5        0.5377             nan     0.1000    0.1814
    ##      6        0.4297             nan     0.1000    0.1409
    ##      7        0.3455             nan     0.1000    0.1115
    ##      8        0.2789             nan     0.1000    0.0887
    ##      9        0.2259             nan     0.1000    0.0711
    ##     10        0.1834             nan     0.1000    0.0573
    ##     20        0.0242             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.4552
    ##      2        1.3246             nan     0.1000    0.3100
    ##      3        1.1322             nan     0.1000    0.2391
    ##      4        0.9841             nan     0.1000    0.1981
    ##      5        0.8621             nan     0.1000    0.1589
    ##      6        0.7631             nan     0.1000    0.1445
    ##      7        0.6743             nan     0.1000    0.1224
    ##      8        0.5997             nan     0.1000    0.1076
    ##      9        0.5340             nan     0.1000    0.0886
    ##     10        0.4787             nan     0.1000    0.0800
    ##     20        0.1648             nan     0.1000    0.0262
    ##     40        0.0237             nan     0.1000    0.0037
    ##     60        0.0039             nan     0.1000    0.0006
    ##     80        0.0008             nan     0.1000    0.0001
    ##    100        0.0003             nan     0.1000    0.0000
    ##    120        0.0001             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7766
    ##      2        1.1446             nan     0.1000    0.4630
    ##      3        0.8698             nan     0.1000    0.3209
    ##      4        0.6789             nan     0.1000    0.2371
    ##      5        0.5376             nan     0.1000    0.1811
    ##      6        0.4297             nan     0.1000    0.1412
    ##      7        0.3454             nan     0.1000    0.1115
    ##      8        0.2788             nan     0.1000    0.0888
    ##      9        0.2258             nan     0.1000    0.0710
    ##     10        0.1833             nan     0.1000    0.0572
    ##     20        0.0242             nan     0.1000    0.0072
    ##     40        0.0006             nan     0.1000    0.0001
    ##     60        0.0001             nan     0.1000   -0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7760
    ##      2        1.1446             nan     0.1000    0.4617
    ##      3        0.8699             nan     0.1000    0.3214
    ##      4        0.6789             nan     0.1000    0.2372
    ##      5        0.5376             nan     0.1000    0.1811
    ##      6        0.4296             nan     0.1000    0.1412
    ##      7        0.3453             nan     0.1000    0.1114
    ##      8        0.2788             nan     0.1000    0.0888
    ##      9        0.2258             nan     0.1000    0.0709
    ##     10        0.1833             nan     0.1000    0.0571
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000    0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.4532
    ##      2        1.3248             nan     0.1000    0.3095
    ##      3        1.1317             nan     0.1000    0.2410
    ##      4        0.9827             nan     0.1000    0.1963
    ##      5        0.8620             nan     0.1000    0.1578
    ##      6        0.7634             nan     0.1000    0.1443
    ##      7        0.6746             nan     0.1000    0.1223
    ##      8        0.5996             nan     0.1000    0.1076
    ##      9        0.5339             nan     0.1000    0.0887
    ##     10        0.4787             nan     0.1000    0.0844
    ##     20        0.1651             nan     0.1000    0.0255
    ##     40        0.0236             nan     0.1000    0.0034
    ##     60        0.0039             nan     0.1000    0.0006
    ##     80        0.0007             nan     0.1000    0.0001
    ##    100        0.0002             nan     0.1000    0.0000
    ##    120        0.0001             nan     0.1000    0.0000
    ##    140        0.0000             nan     0.1000    0.0000
    ##    150        0.0000             nan     0.1000    0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7762
    ##      2        1.1447             nan     0.1000    0.4620
    ##      3        0.8699             nan     0.1000    0.3205
    ##      4        0.6790             nan     0.1000    0.2376
    ##      5        0.5377             nan     0.1000    0.1815
    ##      6        0.4296             nan     0.1000    0.1413
    ##      7        0.3454             nan     0.1000    0.1115
    ##      8        0.2789             nan     0.1000    0.0887
    ##      9        0.2259             nan     0.1000    0.0712
    ##     10        0.1833             nan     0.1000    0.0572
    ##     20        0.0242             nan     0.1000    0.0073
    ##     40        0.0007             nan     0.1000    0.0001
    ##     60        0.0001             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7765
    ##      2        1.1447             nan     0.1000    0.4616
    ##      3        0.8699             nan     0.1000    0.3208
    ##      4        0.6790             nan     0.1000    0.2371
    ##      5        0.5377             nan     0.1000    0.1812
    ##      6        0.4297             nan     0.1000    0.1414
    ##      7        0.3455             nan     0.1000    0.1114
    ##      8        0.2789             nan     0.1000    0.0888
    ##      9        0.2259             nan     0.1000    0.0710
    ##     10        0.1834             nan     0.1000    0.0574
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000
    ##    120        0.0000             nan     0.1000   -0.0000
    ##    140        0.0000             nan     0.1000   -0.0000
    ##    150        0.0000             nan     0.1000   -0.0000
    ## 
    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1        1.6094             nan     0.1000    0.7771
    ##      2        1.1446             nan     0.1000    0.4621
    ##      3        0.8698             nan     0.1000    0.3213
    ##      4        0.6788             nan     0.1000    0.2374
    ##      5        0.5376             nan     0.1000    0.1812
    ##      6        0.4295             nan     0.1000    0.1414
    ##      7        0.3453             nan     0.1000    0.1113
    ##      8        0.2788             nan     0.1000    0.0888
    ##      9        0.2258             nan     0.1000    0.0710
    ##     10        0.1833             nan     0.1000    0.0572
    ##     20        0.0241             nan     0.1000    0.0073
    ##     40        0.0005             nan     0.1000    0.0001
    ##     60        0.0000             nan     0.1000    0.0000
    ##     80        0.0000             nan     0.1000   -0.0000
    ##    100        0.0000             nan     0.1000   -0.0000

We see now how the model performs on the validation dataset:

    prediction.validation <- predict(modelFit, validation)
    conf.matrix <- confusionMatrix(prediction.validation, factor(validation$classe))
    print(conf.matrix)

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1674    0    0    0    0
    ##          B    0 1138    0    0    0
    ##          C    0    1 1026    0    0
    ##          D    0    0    0  964    0
    ##          E    0    0    0    0 1082
    ## 
    ## Overall Statistics
    ##                                      
    ##                Accuracy : 0.9998     
    ##                  95% CI : (0.9991, 1)
    ##     No Information Rate : 0.2845     
    ##     P-Value [Acc > NIR] : < 2.2e-16  
    ##                                      
    ##                   Kappa : 0.9998     
    ##                                      
    ##  Mcnemar's Test P-Value : NA         
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            1.0000   0.9991   1.0000   1.0000   1.0000
    ## Specificity            1.0000   1.0000   0.9998   1.0000   1.0000
    ## Pos Pred Value         1.0000   1.0000   0.9990   1.0000   1.0000
    ## Neg Pred Value         1.0000   0.9998   1.0000   1.0000   1.0000
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2845   0.1934   0.1743   0.1638   0.1839
    ## Detection Prevalence   0.2845   0.1934   0.1745   0.1638   0.1839
    ## Balanced Accuracy      1.0000   0.9996   0.9999   1.0000   1.0000

We can now apply the model to predict the testing dataset:

    prediction.testing <- predict(modelFit, testing)
    print(prediction.testing)

    ##  [1] A A A A A B D B A A A A B A E A A B B B
    ## Levels: A B C D E
