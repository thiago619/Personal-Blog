---
title: "Valor total do CFe maior que o somatório dos meios de pagamento"
date: 2024-07-25T22:56:22-03:00
draft: false
author: "Thiago Santos"
translationKey: "rejeicao-1408"
tags: ['fiscal','pdv','programacão']
---



Um erro muito comum, que ocorre até mesmo em PDVs de grandes empresas ao emitir nota fiscal é a rejeição 1408. Aqui vamos mostrar o que é, por qual motivo ocorre, e como evitar que isso ocorra.

O que é esse erro

De acordo com a documentação oficial da Sefaz, essa é a descrição do erro:

Quando a soma das formas de pagamento(dinheiro, cheque, cartão,ticket, etc) é menor que o valor da nota, a Sefaz rejeita este cupom. E como ela rejeita, ele fica em contigência até que o mesmo tenha o seu valor editado. Ou seja é um erro de sistema, pois o mesmo deveria prevenir o caixa de enviar uma nota inválida. Ou seja, é um tipo de erro que não deveria acontecer, pois o software consegue barrar. Mas se em seu sistema ocorre esse erro, peça urgente para o suporte corrigir,(ou se você for o prestador de serviço e seu PDV tem esse problema leia até o final) pois ele tem poucas soluções e causa um certo transtorno.

O que fazer quando isso ocorre?

O ideal é não acontecer. Mas se você está no meio do movimento e vai tirar um cupom e ele fica em contigência por esse motivo, há duas soluções. Em alguns sistemas, é possível editar os campos da nfe. Então nesse caso, você edita para os meios de pagamento ficarem no mesmo valor e envia a nota. Mas cuidado, pois se você fizer algo errado e enviar a nota terá de cancelar e remarcar tudo de novo. Mas agora, se seu PDVs não permite a edição da nfe,(a grande maioria) você terá de acionar o suporte para eles a editem diretamente no banco de dados.(Imagine a casa cheia, o cliente querendo o cupom e você pendurado no telefone pro suporte emitir o cupom)

O que fazer para isso não ocorrer?

Esta é a parte técnica agora. A menos que você seja um programador, uma empresa de software, ou uma pessoa muito curiosa, recomendo você ficar por aqui.

Um if else

Começando pelo mais simples, um código que compara os valores lançados e o valor da nota. Se o lançamento for menor que o valor da nota, não envie a nota.

if(recebimento<valor) return false;

Armazenando valores monetários corretamente no banco de dados

Muitos bancos de dados armazenam valores monenários como double ou float. O problema é que esses tipos de dados não armazenam o valor exato, mas sim apenas uma aproximação. Se você tem acesso ao banco de dados de um pdv que armazena valores monetários como float ou double, verá bastante valores quebrados como 2,897676 ou 7,7100001 sendo que só temos duas casas decimais para os centavos. Isso impede do banco ou até mesmo da sua aplicação fazer uma comparação de valores precisas, podendo gerar alguns comportamentos indesejados. Uma nelhor escolha para armazenar valores decimais é o decimal.

Calculando rateios da forma correta.

Algumas aplicações permitem o rateio do cupom. Se você fizer uma divisão simples, ocorrerá esse problema, pois irá faltar alguns centavos para chegar. Por exemplo, digamos que seu cliente peça uma bebida que custa R$1,00 e ele quer dividir a nota para 3 pessoas. A aplicação irá dividir o valor em 3, para cada um pagar o mesmo valor. 1 dividido por 3 dá 0,333..., mas a aplicação irá arredondar para R$0,33 centavos. Cada um pagará os 33 centavos e quando você lançar os pagamentos e emitir a nota fiscal, ocorrerá a contigência. Por quê? Porque R$0,33 vezes 3 não dá R$1,00 mas sim R$0,99. E por causa desse um centavo faltando, sua nota vai ficar travada. Para evitar isso, a sua aplicaçao em vez de fazer o rateio a partir da divisão direta, ela pode seguir esse algoritmo:

Multiplica o total por cem.

O valor que der você tira o resto da divisao pela quantidade de rateios.

Divide o resto da divisao por 100

Agora você divide o total da conta e soma o resultado com o resto calculado anteriormente. Ou você soma apenas para uma conta, mas o rateio nao terá valores iguais.

Sendo: T o total da conta e C o número de clientes entao o valor do rateio R será:

R = ((100T)%C)/100) + (T/C)

Em uma calculadora científica fica assim:

((T * 100) mod C) / 100 + (T / C)

Usando essa fórmula para fazer o rateio cada cliente pagará 0,34 centavos e somando tudo dá R$1,02. Mas se você(ou o seu cliente) achar ruim cobrar dois centavos a mais, a sua aplicação pode, aleatoriamente escolher uma conta e somar o fator ((T * 100) mod C) / 100, uma pessoa pagará alguns centavos a mais, mas o valor pago será exatamente o mesmo. No exemplo da bebida, se você usar esse método, um cliente vai pagar 34 centavos e os outros dois pagarão 33 centavos.
