# Desafio Backend — Cassie Stake

## Sumário

[0. Sumário](#sumário)<br/>
[1. Introdução](#introdução)<br/>
[2. Objetivos](#objetivos)<br/>
[3. Arquitetura](#arquitetura)<br/>
[4. Testes](#testes)<br/>
[5. Documentação](#documentação)<br/>
[6. Notas](#notas)

## Introdução

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

#### Cassie Stake é um software de processamento distribuído de apostas. 

A palavra _stake_ do inglês, nesse contexto, significa um envolvimento financial ou interesse pessoal. Já a palavra _Cassie_ se refere a um nome de uma oráculo, que por sua vez, corre riscos ao mudar o futuro. Nessa perspectiva, a junção desses dois termos formam um nome coerente com a proposta de um sistema onde os usuários lidam com riscos futuros.

#### Sobre o uso de IA.

Nesse projeto, o uso de IA até o momento é mínimo, senão zero. O princípio de desenvolvimento é limitar o máximo possível. Além disso, para a validação do cumprimento dessa meta, dados de wakatime serão disponibilizados garantindo que, durante o tempo de desenvolvimento, nenhum código de IA foi implementado. `MISSING_EXTERNAL`

#### Sobre a linguagem

Como a maioria _— senão todos —_ os dados necessários definidos pelas regras de negócio estão em inglês, foi optado por mim por fazer a _codebase_ inteira na mesma linguagem. Entretanto, a documentação seguirá o mesmo padrão da documentação do desafio e permanecerá em português.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

## Objetivos

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

Por causa da relação entre a complexidade do projeto e o tempo disponível, é possível que nem todos os objetivos listados no desafio sejam completos. Por causa disso, este repositório contém uma lista de objetivos e sua complitude disponível [Aqui](./OBJECTIVES.md).

Para mais documenação sobre a forma na qual os pontos descritos foram abordados, funcionam e observações, cheque a [Seção de Detalhes Técnicos](#detalhes-técnicos)

## Arquitetura

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

Como requisitado no desafio, os detalhes de arquitetura estão presentes [Aqui](./ARCHITECTURE.md). Para mais informações de implementação, considere abrir uma _issue_ no repositório do projeto, nesse caso, respostas serão enviadas em até 24h letivas.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

## Testes

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

Teste obrigatório: uma carteira com 100.00 BRL recebe, ao mesmo tempo, duas apostas distintas de 80.00 BRL. O resultado deve conter uma aposta processada, uma rejeição por saldo insuficiente, saldo final de 20.00 BRL e um único débito no ledger. Reenvios não podem alterar esse resultado. Carteiras diferentes devem continuar sendo processadas em paralelo.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

## Documentação

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>

Para aumentar a consistência de conceitos e facilitar a localização de promesas, os arquivos de markdown podem conter as seguintes anotações:

- `MISSING_DOCUMENTATION`: Indica a ausência da documentação de um tópico.
- `MISSING_IMPLEMENTATION`: Indica a ausência de implementação da lógica de negócio descrita.
- `MISSING_EXTERNAL`: Indica a ausência de implementação de um recurso opcional a lógica de negócio.

<div style="text-align: end;"><a href="#sumário">Voltar para o sumário</a></div>
