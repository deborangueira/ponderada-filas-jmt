# Roteiro do vídeo (até 5 min)

Fala em texto corrido, com o que mostrar na tela entre colchetes. Os tempos são aproximados e somam cerca de 4min40s, deixando uma folga.

---

## 1. Abertura (0:00–0:20)

[Tela: modelo aberto no JSIMgraph]

Oi, eu sou a Débora, e neste vídeo vou apresentar a simulação de uma rede de filas que eu construí no JMT. A ideia foi modelar o caminho de uma requisição dentro do sistema de Laboratório do e-Cidade, que é o contexto do projeto do meu grupo neste módulo, para entender quanto tempo o sistema leva para responder, onde está o gargalo e até quanta carga ele aguenta.

## 2. O modelo (0:20–1:30)

[Passar o mouse pelas estações, da esquerda para a direita]

As requisições nascem aqui, nos postos de coleta, chegando a uma taxa de 20 por segundo. Elas passam primeiro por um gateway, que faz autenticação e roteamento, depois por um servidor de aplicação, que aqui tem dois servidores trabalhando em paralelo, e em seguida consultam o banco de dados.

[Apontar a linha de retorno embaixo]

Esta linha de baixo é o que deixa a rede interessante: depois do banco, 40% das requisições voltam para o servidor de aplicação, porque uma mesma operação costuma precisar de mais de uma consulta. Outras 15% são laudos liberados e seguem para um worker que gera o PDF, e as 45% restantes saem do sistema.

[Abrir o Banco de dados → Service Section e depois Routing Section]

Todas as estações atendem por ordem de chegada, com fila infinita e tempo de serviço exponencial. No banco, por exemplo, o tempo médio é de 24 milissegundos, que o JMT mostra como uma taxa de 41,7 atendimentos por segundo, e aqui na aba de roteamento estão as probabilidades de saída. Os valores são hipotéticos, definidos para estudo. Um detalhe importante: por causa do retorno, cada requisição passa em média 1,67 vez pelo banco, então ele recebe na prática 33 consultas por segundo, e não 20.

## 3. Como simulei (1:30–2:10)

[Clicar no ▶ e mostrar a janela de resultados abrindo]

Com o modelo pronto, rodei a simulação por eventos discretos do JMT. Ele simula até conseguir estimar cada medida com 99% de confiança e no máximo 3% de erro, ou até chegar a um milhão de amostras. Fiz isso em duas etapas: primeiro o cenário base, com 20 requisições por segundo, comparando os resultados com o que a teoria de filas prevê, e depois uma análise What-if, aumentando a carga até o sistema saturar.

## 4. Resultados do cenário base (2:10–3:20)

[Aba Utilization]

Na utilização, que é a fração do tempo em que cada servidor fica ocupado, o gateway ficou em 40%, o servidor de aplicação em 66% e o banco de dados em 81%, todos batendo com a teoria. O banco é claramente o gargalo.

[Voltar ao modelo e mostrar as pizzas azuis]

Dá até para ver isso direto no modelo: depois da simulação, o JMT pinta o círculo de cada estação conforme a utilização, e o banco está quase todo azul.

[Aba System Response Time]

O tempo médio de resposta do sistema foi de 0,42 segundo, e a teoria previa 0,43. Fazendo a conta de quanto cada estação contribui, quase metade desse tempo é gasto no banco de dados.

[Aba Number of Customers, mostrando o X vermelho do banco]

E tem um detalhe curioso: o banco foi a única medida que não chegou na precisão pedida. O valor está certo, 4,04 contra 4,00 da teoria, mas uma fila muito ocupada oscila bastante, e por isso precisa de mais tempo de simulação para a média se estabilizar.

## 5. What-if (3:20–4:25)

[Abrir o What-if e mostrar a configuração: Arrival rates, de 20 a 40, 10 passos]

Na análise What-if, repeti a simulação dez vezes, aumentando a taxa de chegada de 20 até 40 requisições por segundo. A teoria dizia que o banco satura em 25.

[Resultados → System Response Time]

E é isso que aparece. Indo de 20 para 22 requisições por segundo, o tempo de resposta sobe de 0,42 para 0,63 segundo. Em 24,4, ele já está em 2,5 segundos, quatro vezes mais, com um aumento de carga parecido. Esse é o joelho da curva: perto do limite, cada pouquinho de carga custa muito tempo. Passando de 25, a fila do banco cresce sem parar, e esses números enormes só mostram até onde a fila chegou antes da simulação acabar.

[Number of Customers → Worker PDF e Servidor de aplicação]

Também dá para ver o efeito em cascata: com o banco travado, o worker de PDF estabiliza, porque o banco só deixa passar uma quantidade fixa, e o servidor de aplicação, que recebe os retornos do banco, acaba saturando também, por volta de 33 requisições por segundo.

## 6. Conclusão (4:25–4:45)

[Voltar ao modelo]

Concluindo, essa arquitetura atende bem a carga atual, mas trabalha perto do limite, e quem define esse limite é o banco de dados. Para crescer, não adianta reforçar o gateway ou a aplicação: é preciso agir no banco, com cache, réplicas de leitura ou diminuindo as consultas repetidas. Obrigada!
