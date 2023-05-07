---
layout: post
title: Github Actions Tip
date: 2023-05-07
categories: [github, actions]
tags: [github actions, tip]
---
# Github Actions Tip
yml파일로 작성

## name
- 가장 상위 name: workflow name
- step 하위 name: step name

## on
action을 실행할 event 정의(push, pull_request)

## jobs
- 각 job을 기준으로 steps을 실행할 container 생성
- job끼리 병렬로 생성

## steps
- -을 기준으로 step이 나눠짐
- 각 step에는 run 또는 uses가 필수
	- uses: 다른 github repository를 사용
	- run: 입력된 명령어를 실행

## 예시
```yml
name: Java CI with Gradle

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'zulu'

      - name: Set up gradle
        uses: gradle/gradle-build-action@v2

      - name: build with gradle
        run: ./gradlew build
```