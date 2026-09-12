# Rastreio de Cor com OpenCV

Este projeto faz o rastreamento de um objeto pela sua cor usando a webcam e OpenCV, em tempo real. A proposta é acompanhar um objeto colorido que se move na frente da câmera e desenhar o rastro do caminho que ele percorreu, aquele efeito de risco que segue a ponta do objeto. Um exemplo prático de uso seria pegar uma tampa azul na mão e movê-la no ar: o programa identifica a tampa pela cor, marca a posição dela quadro a quadro e vai ligando esses pontos, formando a trajetória do movimento na tela.

## Como funciona

O ponto central do projeto é trabalhar no espaço de cor HSV em vez do RGB. Isso é feito de propósito, pois no HSV a informação de cor fica separada da informação de iluminação, o que deixa a detecção bem mais robusta quando a luz do ambiente muda. Cada quadro da webcam é convertido para HSV e, a partir daí, é criada uma máscara que isola apenas os pixels que caem dentro de uma faixa de cor definida, no caso a faixa do azul. O mesmo raciocínio foi montado também para vermelho e verde, bastando trocar qual máscara é usada.

Antes de procurar o objeto, a máscara passa por uma limpeza. Primeiro uma erosão remove os ruídos brancos soltos, aqueles pontinhos que aparecem espalhados e que não são o objeto de verdade. Em seguida uma dilatação devolve ao que sobrou o tamanho original, para não quebrar a forma do objeto que interessa. Com a máscara limpa, o programa busca os contornos presentes e, quando encontra pelo menos um, seleciona o de maior área, partindo da ideia de que o maior contorno daquela cor é justamente o objeto que se quer rastrear. O centro desse contorno é guardado numa fila e a cada quadro os pontos guardados são ligados, gerando o rastro do movimento.

Vale citar a escolha da estrutura que guarda os pontos do rastro. Foi usada uma deque com tamanho máximo fixo, que é uma fila que descarta automaticamente o elemento mais antigo quando um novo entra depois de cheia. Na prática isso significa que o rastro tem um comprimento limitado e vai se apagando pela cauda conforme o objeto avança, sem que seja preciso ficar limpando a lista na mão.

## Técnicas usadas

O projeto reúne várias operações clássicas de visão computacional num fluxo só: conversão de espaço de cor para HSV, segmentação por faixa de cor com máscara, operações morfológicas de erosão e dilatação para limpar ruído, detecção de contornos e seleção do contorno de maior área. Não há modelo de machine learning aqui, o reconhecimento é feito inteiramente por processamento de imagem, o que torna o programa leve e capaz de rodar em tempo real sem precisar de treino.

## Como rodar

Instale as dependências:

```bash
pip install -r requirements.txt
```

Rode o rastreio:

```bash
python rastreio_cor.py
```

A janela do OpenCV abre a webcam e a tecla ESC encerra a execução. Por padrão o rastreio está calibrado para a cor azul. Para rastrear outra cor, basta ajustar no código os limites da faixa HSV correspondente e usar a máscara daquela cor na etapa de detecção de contornos.

## Estrutura do projeto

```
rastreio-cor-opencv/
├── rastreio_cor.py     # rastreamento por cor em tempo real
├── requirements.txt
└── .gitignore
```

## Observações

Os valores de faixa HSV que definem cada cor foram calibrados para condições específicas de iluminação, então em ambientes muito diferentes pode ser necessário reajustá-los. Uma evolução natural seria criar um pequeno painel com barras deslizantes para calibrar a faixa de cor ao vivo, sem ter que mexer no código toda vez que a luz muda.
