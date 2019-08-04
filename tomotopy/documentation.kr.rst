tomotopy 란?
------------------
`tomotopy`는 토픽 모델링 툴인 `tomoto`의 Python 확장 버전입니다. `tomoto`는 c++로 작성된 깁스 샘플링 기반의 토픽 모델링 라이브러리로,
최신 CPU의 벡터화 기술을 활용하여 처리 속도를 최대로 끌어올렸습니다.
현재 버전의 `tomoto`에서는 다음과 같은 주요 토픽 모델들을 지원하고 있습니다.

* Latent Dirichlet Allocation (`tomotopy.LDAModel`)
* Dirichlet Multinomial Regression (`tomotopy.DMRModel`)
* Hierarchical Dirichlet Process (`tomotopy.HDPModel`)
* Multi Grain LDA (`tomotopy.MGLDAModel`)
* Pachinko Allocation (`tomotopy.PAModel`)
* Hierarchical PA (`tomotopy.HPAModel`)

시작하기
---------------
다음과 같이 pip를 이용하면 tomotopy를 쉽게 설치할 수 있습니다.
::

    $ pip install tomotopy

Linux에서는 c++14 코드를 컴파일하기 위해 gcc 5 이상이 필수적으로 설치되어 있어야 합니다. 
설치가 끝난 뒤에는 다음과 같이 Python3에서 바로 import하여 tomotopy를 사용할 수 있습니다.
::

    import tomotopy as tp
    print(tp.isa) # 'avx2'나 'avx', 'sse2', 'none'를 출력합니다.

현재 tomotopy는 가속을 위해 AVX2, AVX or SSE2 SIMD 명령어 세트를 활용할 수 있습니다.
패키지가 import될 때 현재 환경에서 활용할 수 있는 최선의 명령어 세트를 확인하여 최상의 모듈을 자동으로 가져옵니다.
만약 `tp.isa`가 `none`이라면 현재 환경에서 활용 가능한 SIMD 명령어 세트가 없는 것이므로 훈련에 오랜 시간이 걸릴 수 있습니다.
그러나 최근 대부분의 Intel 및 AMD CPU에서는 SIMD 명령어 세트를 지원하므로 SIMD 가속이 성능을 크게 향상시킬 수 있을 것입니다.

간단한 예제로 'sample.txt' 파일로 LDA 모델을 학습하는 코드는 다음과 같습니다.
::

    import tomotopy as tp
    mdl = tp.LDAModel(k=20)
    for line in open('sample.txt'):
        mdl.add_doc(line.strip().split())
    
    for i in range(0, 100, 10):
        mdl.train(10)
        print('Iteration: {}\tLog-likelihood: {}'.format(i, mdl.ll_per_word))
    
    for k in range(mdl.k):
        print('Top 10 words of topic #{}'.format(k))
        print(mdl.get_topic_words(k, top_n=10))

tomotopy의 성능
-----------------------
`tomotopy`는 주제 분포와 단어 분포를 추론하기 위해 Collapsed Gibbs-Sampling(CGS) 기법을 사용합니다.
일반적으로 CGS는 [gensim의 LdaModel]가 이용하는 Variational Bayes(VB) 보다 느리게 수렴하지만 각각의 반복은 빠르게 계산 가능합니다.
게다가 `tomotopy`는 멀티스레드를 지원하므로 SIMD 명령어 세트뿐만 아니라 다중 코어 CPU의 장점까지 활용할 수 있습니다. 이 덕분에 각각의 반복이 훨씬 빠르게 계산 가능합니다.

[gensim의 LdaModel]: https://radimrehurek.com/gensim/models/ldamodel.html 

다음의 차트는 `tomotopy`와 `gensim`의 LDA 모형 실행 시간을 비교하여 보여줍니다.
입력 문헌은 영어 위키백과에서 가져온 1000개의 임의 문서이며 전체 문헌 집합은 총 1,506,966개의 단어로 구성되어 있습니다. (약 10.1 MB).
`tomotopy`는 200회를, `gensim` 10회를 반복 학습하였습니다.

.. image:: https://bab2min.github.io/tomotopy/images/tmt_i5.png

Intel i5-6600, x86-64 (4 cores)에서의 성능

.. image:: https://bab2min.github.io/tomotopy/images/tmt_xeon.png

Intel Xeon E5-2620 v4, x86-64 (8 cores, 16 threads)에서의 성능

`tomotopy`가 20배 더 많이 반복하였지만 전체 실행시간은 `gensim`보다 5~10배 더 빨랐습니다. 또한 `tomotopy`는 전반적으로 안정적인 결과를 보여주고 있습니다.

CGS와 VB는 서로 접근방법이 아예 다른 기법이기 때문에 둘을 직접적으로 비교하기는 어렵습니다만, 실용적인 관점에서 두 기법의 속도와 결과물을 비교해볼 수 있습니다.
다음의 차트에는 두 기법이 학습 후 보여준 단어당 로그 가능도 값이 표현되어 있습니다.

.. image:: https://bab2min.github.io/tomotopy/images/LLComp.png

<table style='width:100%'>
<tbody><tr><th colspan="2">`tomotopy`가 생성한 주제들의 상위 단어</th></tr>
<tr><th>#1</th><td>use, acid, cell, form, also, effect</td></tr>
<tr><th>#2</th><td>use, number, one, set, comput, function</td></tr>
<tr><th>#3</th><td>state, use, may, court, law, person</td></tr>
<tr><th>#4</th><td>state, american, nation, parti, new, elect</td></tr>
<tr><th>#5</th><td>film, music, play, song, anim, album</td></tr>
<tr><th>#6</th><td>art, work, design, de, build, artist</td></tr>
<tr><th>#7</th><td>american, player, english, politician, footbal, author</td></tr>
<tr><th>#8</th><td>appl, use, comput, system, softwar, compani</td></tr>
<tr><th>#9</th><td>day, unit, de, state, german, dutch</td></tr>
<tr><th>#10</th><td>team, game, first, club, leagu, play</td></tr>
<tr><th>#11</th><td>church, roman, god, greek, centuri, bc</td></tr>
<tr><th>#12</th><td>atom, use, star, electron, metal, element</td></tr>
<tr><th>#13</th><td>alexand, king, ii, emperor, son, iii</td></tr>
<tr><th>#14</th><td>languag, arab, use, word, english, form</td></tr>
<tr><th>#15</th><td>speci, island, plant, famili, order, use</td></tr>
<tr><th>#16</th><td>work, univers, world, book, human, theori</td></tr>
<tr><th>#17</th><td>citi, area, region, popul, south, world</td></tr>
<tr><th>#18</th><td>forc, war, armi, militari, jew, countri</td></tr>
<tr><th>#19</th><td>year, first, would, later, time, death</td></tr>
<tr><th>#20</th><td>apollo, use, aircraft, flight, mission, first</td></tr>
</tbody></table>


<table style='width:100%'>
<tbody><tr><th colspan="2">`gensim`이 생성한 주제들의 상위 단어</th></tr>
<tr><th>#1</th><td>use, acid, may, also, azerbaijan, cell</td></tr>
<tr><th>#2</th><td>use, system, comput, one, also, time</td></tr>
<tr><th>#3</th><td>state, citi, day, nation, year, area</td></tr>
<tr><th>#4</th><td>state, lincoln, american, war, union, bell</td></tr>
<tr><th>#5</th><td>anim, game, anal, atari, area, sex</td></tr>
<tr><th>#6</th><td>art, use, work, also, includ, first</td></tr>
<tr><th>#7</th><td>american, player, english, politician, footbal, author</td></tr>
<tr><th>#8</th><td>new, american, team, season, leagu, year</td></tr>
<tr><th>#9</th><td>appl, ii, martin, aston, magnitud, star</td></tr>
<tr><th>#10</th><td>bc, assyrian, use, speer, also, abort</td></tr>
<tr><th>#11</th><td>use, arsen, also, audi, one, first</td></tr>
<tr><th>#12</th><td>algebra, use, set, ture, number, tank</td></tr>
<tr><th>#13</th><td>appl, state, use, also, includ, product</td></tr>
<tr><th>#14</th><td>use, languag, word, arab, also, english</td></tr>
<tr><th>#15</th><td>god, work, one, also, greek, name</td></tr>
<tr><th>#16</th><td>first, one, also, time, work, film</td></tr>
<tr><th>#17</th><td>church, alexand, arab, also, anglican, use</td></tr>
<tr><th>#18</th><td>british, american, new, war, armi, alfr</td></tr>
<tr><th>#19</th><td>airlin, vote, candid, approv, footbal, air</td></tr>
<tr><th>#20</th><td>apollo, mission, lunar, first, crew, land</td></tr>
</tbody></table>

어떤 SIMD 명령어 세트를 사용하는지는 성능에 큰 영향을 미칩니다.
다음 차트는 SIMD 명령어 세트에 따른 성능 차이를 보여줍니다.

.. image:: https://bab2min.github.io/tomotopy/images/SIMDComp.png

다행히도 최신 x86-64 CPU들은 대부분 AVX2 명령어 세트를 지원하기 때문에 대부분의 경우 AVX2의 높은 성능을 활용할 수 있을 것입니다.

모델의 저장과 불러오기
-------------------
`tomotopy`는 각각의 토픽 모델 클래스에 대해 `save`와 `load` 메소드를 제공합니다.
따라서 학습이 끝난 모델을 언제든지 파일에 저장하거나, 파일로부터 다시 읽어와서 다양한 작업을 수행할 수 있습니다.
::

    import tomotopy as tp
    
    mdl = tp.HDPModel()
    for line in open('sample.txt'):
        mdl.add_doc(line.strip().split())
    
    for i in range(0, 100, 10):
        mdl.train(10)
        print('Iteration: {}\tLog-likelihood: {}'.format(i, mdl.ll_per_word))
    
    # 파일에 저장
    mdl.save('sample_hdp_model.bin')
    
    # 파일로부터 불러오기
    mdl = tp.HDPModel.load('sample_hdp_model.bin')
    for k in range(mdl.k):
        if not mdl.is_live_topic(k): continue
        print('Top 10 words of topic #{}'.format(k))
        print(mdl.get_topic_words(k, top_n=10))
    
    # 저장된 모델이 HDP 모델이었기 때문에, 
    # LDA 모델에서 이 파일을 읽어오려고 하면 예외가 발생합니다.
    mdl = tp.LDAModel.load('sample_hdp_model.bin')

파일로부터 모델을 불러올 때는 반드시 저장된 모델의 타입과 읽어올 모델의 타입이 일치해야합니다.

이에 대해서는 `tomotopy.LDAModel.save`와 `tomotopy.LDAModel.load`에서 더 자세한 내용을 확인할 수 있습니다.

모델 안의 문헌과 모델 밖의 문헌
-------------------------------------------
토픽 모델은 크게 2가지 목적으로 사용할 수 있습니다. 
기본적으로는 문헌 집합으로부터 모델을 학습하여 문헌 내의 주제들을 발견하기 위해 토픽 모델을 사용할 수 있으며,
더 나아가 학습된 모델을 활용하여 학습할 때는 주어지지 않았던 새로운 문헌에 대해 주제 분포를 추론하는 것도 가능합니다.
전자의 과정에서 사용되는 문헌(학습 과정에서 사용되는 문헌)을 **모델 안의 문헌**,
후자의 과정에서 주어지는 새로운 문헌(학습 과정에 포함되지 않았던 문헌)을 **모델 밖의 문헌**이라고 가리키도록 하겠습니다.

`tomotopy`에서 이 두 종류의 문헌을 생성하는 방법은 다릅니다. **모델 안의 문헌**은 `tomotopy.LDAModel.add_doc`을 이용하여 생성합니다.
add_doc은 `tomotopy.LDAModel.train`을 시작하기 전까지만 사용할 수 있습니다. 
즉 train을 시작한 이후로는 학습 문헌 집합이 고정되기 때문에 add_doc을 이용하여 새로운 문헌을 모델 내에 추가할 수 없습니다.

또한 생성된 문헌의 인스턴스를 얻기 위해서는 다음과 같이 `tomotopy.LDAModel.docs`를 사용해야 합니다.

::

    mdl = tp.LDAModel(k=20)
    idx = mdl.add_doc(words)
    if idx < 0: raise RuntimeError("Failed to add doc")
    doc_inst = mdl.docs[idx]
    # doc_inst is an instance of the added document

**모델 밖의 문헌**은 `tomotopy.LDAModel.make_doc`을 이용해 생성합니다. make_doc은 add_doc과 반대로 train을 시작한 이후에 사용할 수 있습니다.
만약 train을 시작하기 전에 make_doc을 사용할 경우 올바르지 않은 결과를 얻게 되니 이 점 유의하시길 바랍니다. make_doc은 바로 인스턴스를 반환하므로 반환값을 받아 바로 사용할 수 있습니다.

::

    mdl = tp.LDAModel(k=20)
    # add_doc ...
    mdl.train(100)
    doc_inst = mdl.make_doc(unseen_words) # doc_inst is an instance of the unseen document

새로운 문헌에 대해 추론하기
------------------------------
`tomotopy.LDAModel.make_doc`을 이용해 새로운 문헌을 생성했다면 이를 모델에 입력해 주제 분포를 추론하도록 할 수 있습니다. 
새로운 문헌에 대한 추론은 `tomotopy.LDAModel.infer`를 사용합니다.

::

    mdl = tp.LDAModel(k=20)
    # add_doc ...
    mdl.train(100)
    doc_inst = mdl.make_doc(unseen_words)
    topic_dist, ll = mdl.infer(doc_inst)
	print("Topic Distribution for Unseen Docs: ", topic_dist)
    print("Log-likelihood of inference: ", ll)

infer 메소드는 `tomotopy.Document` 인스턴스 하나를 추론하거나 `tomotopy.Document` 인스턴스의 `list`를 추론하는데 사용할 수 있습니다. 
자세한 것은 `tomotopy.LDAModel.infer`을 참조하길 바랍니다.

라이센스
---------
`tomotopy`는 MIT License 하에 배포됩니다.