### task
Проблема: graphviz online + URL + image

Нужно в online (server) передавать на рендеринг код dot (graphviz) как передача параметра URL.
Так умеют делать такие graphviz online: dreampuf.github.io/GraphvizOnline и www.devtoolsdaily.com/graphviz

Другие, например, https://edotor.net/ и https://magjac.com/graphviz-visual-editor/ не позволяют в параметре URL передать код (схему).

Однако указанные:   
dreampuf.github.io/GraphvizOnline и www.devtoolsdaily.com/graphviz не позволяют рендерить картинки с тегом image="
Например, см. https://github.com/bpmbpm/family-tree/blob/main/ver2/graphviz_online_URL_image_v1.md

Анализ причин был выполнен в https://github.com/bpmbpm/family-tree/pull/30 см. также https://github.com/bpmbpm/family-tree/blob/main/design/problem.md#pull-30

Задачи:
1. произведи анализ graphviz online, которые одновременно могут передавать код dot в параметром URL и отображать изображения через тег image="
2. в папке ver1 сохрани код в index.html, реализующий graphviz online, но при обеспечении условия, указанного выше (graphviz online + URL + image). Приведи тестовый пример (dot - код приведи в отдельном текстовом файле).  
Требования см. https://github.com/bpmbpm/graphviz-online/blob/main/requirements/programming_information.md
