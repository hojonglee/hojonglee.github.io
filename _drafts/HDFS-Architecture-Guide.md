---
layout: post
title: HDFS Architecture Guide
author: 이호종
categories: [dev]
comments: true

---

## Introduction

하둡 분산 파일 시스템(Hadoop Distributed File System, HDFS)은 범용 하드웨어에서 실행되도록 설계된 분산 파일 시스템이다. 이미 존재하는 파일 시스템과 매우 유사하다. 그러나 다른 분산 파일 시스템과 중요한 차이가 있다. HDFS는 높은 fault-tolerant를 가지고 있고 저비용 하드웨어에 배포할 수 있게 설계되어있다. HDFS는 어플리케이션 데이터에 높은 처리량의 액세스를 제공하고 대용량 데이터 셋이 있는 어플리케이션에 적합하다. HDFS는 파일 시스템 데이터에 대한 스프리밍 액세스를 가능하게 하는 몇 가지 POSIX 요구사항을 완화한다. HDFS는 원래 Apache Nutch 웹 검색 엔진 프로젝트에 대한 기반으로 설계되었다. HDFS는 지금 Apache Hadoop의 하위 프로젝트이다. 프로젝트 URL은 [http://hadoop.apache.org/hdfs](http://hadoop.apache.org/hdfs) 이다.

## Assumptions and Goals

### Hardware Failure

하드웨어 오류는 예외가 아니라 표준이다. HDFS 인스턴스는 수백 혹은 수천개의 서버로 구성되며 각각은 파일 시스템의 데이터 일부를 저장한다. 엄청난 수의 구성요소가 있고 각 구성요소는 오류가 발생할 확률이 non-trivial 이라는 것은 HDFS의 일부 구성요소는 항상 동작하지 않는다는 것을 의미한다. 그러므로 오류의 발견과 빠른 자동 복구 기능은 HDFS의 핵심 구조의 목표이다.

> non-trivial은 선형대수학에서 0행렬 이외의 해를 갖는 것을 말합니다. [이곳](http://blog.naver.com/nakta80/102117770)을 참고했습니다.

### Streaming Data Access

HDFS에서 수행되는 어플리케이션은 데이터 셋에 대한 스트리밍 액세스가 필요하다. 일반적인 목적의 파일 시스템에서 수행되는 일반적인 목적의 어플리케이션이 아니다. HDFS는 사용자가 대화식으로 사용하기보다는 배치처리를 위해 설계되었다. 강조점은 데이터 액세스에 대한 낮은 지연시간이 아니라 데이터 액세스에 대한 높은 처리량이다. POSIX는 HDFS에서 사용되는 어플리케이션에 필요하지 않은 많은 어려운 요구사항을 부과한다. 몇 가지 주요 영역에서 POSIX 의미론은 데이터 처리 속도를 높이기 위해 거래되었다.

### Large Data Sets

HDFS에서 수행되는 애플리케이션은 큰 데이터 셋을 갖는다. HDFS의 일반적인 파일은 크기가 기가바이트에서 테라바이트에 이른다. 그러므로, HDFS는 큰 파일을 지원하도록 조정되었다. 큰 집합 데이터 대역폭을 제공하고 단일 클러스터에서 수백개의 노드로 확장한다. 단일 인스턴스에서 수천만의 파일을 지원한다.

### Simple Coherency Model

HDFS 애플리케이션은 파일에 대하여 한번 쓰고 많이 읽는(WORM, write-once-read-many) 액세스 모델을 필요로 한다. 일단 생성되어 작성되고 닫힌 파일은 추가(append) 및 절단(truncate)을 제외하고 변경될 필요가 없다. 파일의 끝에 내용을 추가하는 것은 지원되지만 임의의 위치에 수정할 수 없다. 이 가정은 데이터 일관성 이슈를 간단하게하고 데이터 액세스의 처리량을 높여준다. 맵-리듀스 어플리케이션 또는 웹 수집기 어플리케이션에 이 모델이 정확히 맞는다.

### "Moving Computation is Cheaper than Moving Data"

애플리케이션에게 요청받은 계산은 동작하는 데이터 근처에서 실행되면 훨씬 더 효율적이다. 데이터 셋의 크기가 클때 특별히 그렇다. 네트워크 점유를 최소화 하고 시스템의 전체 처리량을 늘린다. 애플리케이션이 수행되는 곳으로 데이터를 옮기는 것보다 데이터가 있는 곳에서 계산하고 합치는 것이 더 낫다. HDFS는 애플리케이션에 대하여 데이터가 위치한 곳으로 옮길 수 있는 인터페이스를 제공한다.

### Portability Across Heterogeneous Hardware and Software Platforms

HDFS는 한 플랫폼에서 다른 플랫폼으로 이식하기 쉽게 설계되어있다. 이를 통해 HDFS를 다양한 애플리케이션에 적합한 플랫폼으로 널리 채택 할 수 있다.

## NameNode and DataNodes

HDFS는 master/slave 구조를 가진다. HDFS 클러스터는 파일 시스템 네임스페이스를 관리하고 클라이언트가 파일에 접근하는 것을 제어하는 마스터 서버인 단일 NameNode로 구성된다. 또한 일반적으로 클러스터에 노드당 하나의 DataNode가 있다. 이 노드는 동작하는 노드에 연결된 저장소를 관리한다. 
































