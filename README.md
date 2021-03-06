# Spark를 이용한 분류 실습 (iris 데이터 분류하기)



## Spark 내려 받기

- windows terminal or mac terminal을 연다. 
- 소스 코드를 다운 받을 적당한 디렉토리로 이동한다.  
- git 명령어를 이용해 Spark를 내려 받는다. 

```
git clone https://github.com/CUKykkim/data_preprocessing.git
```


## Spark 수행하기

- git을 통해 다운 받은 디렉토리 안으로 들어간다. 

```
cd data_preprocessing
```

- `docker-compose` 명령어를 이용해 스파크 컨테이너를 띄운다. 
  
```
docker-compose up
```

- 컨테이너가 모두 수행이 되면 컨테이너는 다음과 같은 상태가 됨

```
docker ps
```

```
CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS          PORTS
                   NAMES
edb3f8d728c9   ykkim77/spark-worker-1   "/bin/bash /worker.sh"   42 seconds ago       Up 36 seconds   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp
                   spark-worker-1
349f58d01f67   ykkim77/spark-master     "/bin/bash /master.sh"   About a minute ago   Up 39 seconds   0.0.0.0:7077->7077/tcp, :::7077->7077/tcp, 6066/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   spark-master
```


- terminal 탭을 하나 더 열어, spark-master 컨테이너로 진입

```
docker exec -it spark-master /bin/bash
```

- spark가 설치된 경로의 디렉토리로 이동

```
cd  ~/../spark/bin/
```


- python으로 spark을 연산을 수행할 수 있는 스파크쉘 수행

```
./pyspark
```



```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 3.1.1
      /_/

Using Python version 3.7.10 (default, Mar  2 2021 09:06:08)
Spark context Web UI available at http://349f58d01f67:4040
Spark context available as 'sc' (master = local[*], app id = local-1632992284633).
SparkSession available as 'spark'.
```



## iris 데이터 전처리 하기


```
from pyspark.ml.feature import StandardScaler,VectorAssembler, StringIndexer
from pyspark.ml.classification import DecisionTreeClassifier, RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

iris = spark.read.csv('iris.csv', header = True, inferSchema = True)  //데이터셋 로드
iris.printSchema()     // 데이터셋의 스키마 살펴보기
iris.show()             // 데이터셋 보기


// 피쳐 벡터 어셈블
df_assembler = VectorAssembler(
    inputCols = ["sepal_length","sepal_width","petal_length","petal_width"], outputCol = 'features').transform(iris)

df_assembler_labelIndexer = StringIndexer(inputCol="species", outputCol="indexedLabel").fit(df_assembler).transform(df_assembler)   // 라벨을 인덱싱

df_assembler_labelIndexer_scaled = StandardScaler(inputCol="features", outputCol="scaled_feature").fit(df_assembler_labelIndexer).transform(df_assembler_labelIndexer)     // // 평균 0, 표준편차 1로 피 정규화

df_assembler_labelIndexer_scaled.show(truncate=False)

(trainingData, testData) = df_assembler_labelIndexer_scaled.randomSplit([0.7, 0.3])    // 학습 데이터와 테스트 데이터로 분리

```



## 의사결정트리 


```
dt = DecisionTreeClassifier(labelCol="indexedLabel", featuresCol="scaled_feature", maxDepth = 3)

dtModel = dt.fit(trainingData)

predictions = dtModel.transform(testData)

predictions.select("prediction", "indexedLabel", "features").show(100)


evaluator = MulticlassClassificationEvaluator(
    labelCol="indexedLabel", predictionCol="prediction", metricName="accuracy")
accuracy = evaluator.evaluate(predictions)

print(accuracy)

```



## 랜덤 포레스트

```

dt = RandomForestClassifier(labelCol="indexedLabel", featuresCol="scaled_feature")

dtModel = dt.fit(trainingData)

predictions = dtModel.transform(testData)

predictions.select("prediction", "indexedLabel", "features").show(100)


evaluator = MulticlassClassificationEvaluator(
    labelCol="indexedLabel", predictionCol="prediction", metricName="accuracy")
accuracy = evaluator.evaluate(predictions)

print(accuracy)

```


