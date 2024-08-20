---
layout: post
title: "Generating structured data from an image with GPT-vision model and LangChain"
date: 2024-08-20 22:00:00 +0900
last_modified_at: 2024-08-21 02:00:00 +0900
categories: [practice]
tags: [LLM, LangChain]
toc: true
comments: true
published: true
---

최근 멀티모달 입력을 처리하는 능력을 갖춘 거대 언어 모델(**LLM**)을 활용하여 이미지, 영상, 음성 데이터 등 다양한 모달리티에서 '의미있는' 정보를 구조화된 형태로 추출하는 작업이 많은 주목을 받고 있다. 이 글에서는 개발자가 특정 요구 사항이나 작업에 맞춰 파이프라인, 에이전트, 워크플로우를 손쉽게 만들 수 있도록 지원하는 **LangChain** 프레임워크를 활용하여 이미지로부터 구조화된 정보를 출력하는 간단한 튜토리얼을 소개한다.

## Preparing the OpenAI API Key
이 튜토리얼을 따라가려면 먼저 LangChain을 설치하고, OpenAI API Key를 가져와 환경변수 또는 코드에서 직접 키를 설정해야 한다.

``` bash
pip install langchain langchain_openai
```

LangChain에서 OpenAI의 ChatGPT 모델을 사용하려면 API Key를 얻어야 한다. API Key는 다음의 OpenAI 웹페이지에서 API 가이드를 따르면 된다. [[OpenAI developer platform]](https://platform.openai.com/)

API 키가 있다면, 시스템에서 환경변수로 설정하거나 코드에서 직접 제공할 수 있다.
``` bash
export OPENAI_API_KEY="your_openai_api_key_here"
```
또는,

``` python
# config.py
import os
import langchain
os.environ["OPENAI_API_KEY"] = "your_openai_api_key_here"
```
## Encoding the Image
LangChain을 사용하여 이미지를 처리하기 전에, 이미지 데이터를 언어 모델에 전달할 수 있는 형식으로 인코딩해야 한다. 아래 코드는 `load_image`라는 함수를 정의하여 주어진 경로(`image_path`)에서 이미지를 불러와 `base64` 문자열로 인코딩하고, 이를 `image` 키를 포함한 사전 형태로 반환한다.

``` python
# image_loader.py
import base64

def load_image(inputs: dict) -> dict:
    """Load image from file and encode it as base64."""
    image_path = inputs["image_path"]

    def encode_image(image_path):
        with open(image_path, "rb") as image_file:
            return base64.b64encode(image_file.read()).decode('utf-8')
 
    image_base64 = encode_image(image_path)
    return {"image": image_base64}
```
## Defining the Output Structure
이미지에서 정보를 추출하기 전에, 출력 구조를 정의해야 한다. 아래 코드는 이미지에 대한 설명(`image_description`)과 함께 추출하고자 하는 추가 정보에 대한 필드(`people_count`, `main_objects`)를 포함하는 `ImageInformation`이라는 Pydantic 모델을 정의하는 예시이다. 이 모델은 LangChain의 `JsonOutputParser`을 불러와 모델의 출력 결과를 지정된 구조로 파싱을 수행합니다.

``` python
# output_parser.py
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_core.output_parsers import JsonOutputParser

class ImageInformation(BaseModel):
    image_description: str = Field(description="a short description of the image")
    people_count: int = Field(description="number of humans on the picture")
    main_objects: list[str] = Field(description="list of the main objects on the picture")

parser = JsonOutputParser(pydantic_object=ImageInformation)
```

## Setting up the Image Model
다음은 `ChatOpenAI` 클래스를 사용하여 텍스트와 이미지 데이터를 처리할 수 있는 모델인 `gpt-4o-mini`를 호출하고 `HumanMessage` 객체를 생성하여 텍스트 프롬프트, 출력 형식 지시문, 인코딩된 이미지 데이터를 모델에 전달한 후 응답을 반환하는 간단한 코드 구성이다.

``` python
# image_model.py
import langchain
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI
from langchain_core.runnables import chain
from config import os
from output_parser import parser

@chain
def image_model(inputs: dict) -> str | list[str] | dict:
    """
    temperature: Controls the randomness of the model's output; lower values produce more deterministic responses. (0 ~ 1)
    model: Specifies the model name to be used for processing text and image inputs.
    max_tokens: Sets the maximum number of tokens (words and symbols) that the model can generate in the response.
    """
    model = ChatOpenAI(
        temperature=0.5, 
        model="gpt-4o-mini",  
        max_tokens=1024 
    )
    msg = model.invoke(
        [HumanMessage(
            content=[
                {"type": "text", "text": inputs["prompt"]},
                {"type": "text", "text": parser.get_format_instructions()},
                {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{inputs['image']}"}},
            ]
        )]
    )

    return msg.content
```
## Running the Entire Process
마지막으로 이미지 파일을 로드하여 GPT 모델을 통해 분석하고, 이미지에 대한 구조화된 정보(*여기서는 이미지 안에서 사람의 수, 주요 객체 리스트, 이미지에 대한 설명을 포함*)를 추출하고 결과를 출력하는 코드를 구성한다. 이때, `TransformChain`은 이미지 인코딩 작업을 체인의 일부로 정의하며, 이를 다른 체인들과 연결해 복잡한 워크플로우를 구성할 수 있도록 한다.

``` python
# main.py
from image_loader import load_image
from image_model import image_model
from output_parser import parser
from langchain.chains import TransformChain
from langchain import globals

# Set verbose
globals.set_debug(True)

def get_image_informations(image_path: str) -> dict:
    vision_prompt = """
   Given the image, provide the following information:
   - A count of how many people are in the image
   - A list of the main objects present in the image
   - A description of the image
"""
    load_image_chain = TransformChain(
        input_variables=["image_path"],
        output_variables=["image"],
        transform=load_image
    )
    # Combine the image loading, model invocation, and output parsing into one chain
    vision_chain = load_image_chain | image_model | parser

    return vision_chain.invoke({'image_path': f'{image_path}',
                               'prompt': vision_prompt})

result = get_image_informations("path/to/your/image.jpg")
print(result)
```

이미지를 입력하여 `main.py`를 실행한 결과는 다음과 같다.

![Result](/assets/img/practice/2024-08-20/result.png){: width="100%" }

## Conclusion
위 튜토리얼에서는 **LangChain**의 기능을 활용하여, 언어 모델과 사용자 정의 프롬프트를 통해 이미지에서 구조화된 정보를 추출하는 방법을 다루었다. 앞선 예제에서는 이미지로부터 정보를 꽤 성공적으로 추출하였지만, 더 복잡한 이미지를 입력으로 넣거나 고차원적인 추론을 요구하였을 때 잘못된 결과가 출력됨을 확인할 수 있었다. 이러한 문제를 해결하고 더 나은 출력 품질을 얻기 위해서는 더 높은 성능을 갖춘 LLM 모델을 활용하거나, 보다 정교한 프롬프트를 사용하는 것이 더욱 중요해진다. 앞으로 언어 모델을 적용함에 있어서 최적화 된 솔루션을 찾기 위해 도메인에 특화된 언어 모델을 적용하거나 프롬프트 엔지니어링을 공부하고자 한다.

## References
[1] [LangChain Docs](https://python.langchain.com/v0.2/docs/introduction/) <br>
[2] [OpenAI ChatGPT Vision Guide](https://platform.openai.com/docs/guides/vision) <br>
[3] [Generating structured data from an image with GPT vision and Langchain, Benoit Pothier, Medium](https://medium.com/@bpothier/generating-structured-data-from-an-image-with-gpt-vision-and-langchain-34aaf3dcb215)