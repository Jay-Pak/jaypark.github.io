---
layout: post
title:  "Google Cloud Platform에서 Jupiter Notebook을 이용하는 새로운 방법"
date:   2019-06-05 00:18:23 +0900
categories: [google-cloud,big-data,python,compute-engine,ml
---

Google Cloud에서 기존에는 Datalab을 통해 Python을 이용하는 Instance를 이용하거나, Compute Engine Instance에 Jupyter Notebook을 설치 하여 이용 해야 했다. 하지만, GPU를 이용 하거나, 기타 Library(tensorflow,pytorch etc.)를 이용하기 위해서는 따로 설치을 해야 했고, 또한 사용하기 위해 설정 해야 되는 것들이 많이 있었다.

물론 Kubeflow에서 Jupyter Notebook Pod를 사용할 수 있는 기능이 존재 하지만, 이 또한 GPU Driver를 사용하기 위한 설정이 추가로 필요한 것이 사실이다. Google AI Platform에서는 모든 것이 사전에 정의된 Service를 제공하여, 사용자 편의성을 높였다. 지금부터는 