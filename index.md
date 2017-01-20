---
title: Introdução
date: '2017-01-20'
---





> "(...) The fact that data science exists as a field is a colossal failure of statistics. To me, [what I do] is what statistics is all about. It is gaining insight from data using modelling and visualization. Data munging and manipulation is hard and statistics has just said that’s not our domain."
> 
> Hadley Wickham

## Pacotes `dplyr` e `tidyr`

A transformação de dados é uma tarefa dolorosa e demorada, tomando muitas vezes a maior parte do tempo de uma análise estatística.

O `dplyr` é um dos pacotes mais úteis para realizar transformação de dados, aliando simplicidade e eficiência de uma forma elegante. Os scripts em `R` que fazem uso inteligente dos verbos `dplyr` e as facilidades do operador _pipe_ tendem a ficar mais legíveis e organizados, sem perder velocidade de execução.

Por ser um pacote que se propõe a realizar um dos trabalhos mais árduos da análise estatística, e por atingir esse objetivo de forma elegante, eficaz e eficiente, o `dplyr` pode ser considerado como uma revolução no `R`.

### Trabalhando com `tibble`s

A `tibble` nada mais é do que um `data.frame`, mas com um método de impressão mais adequado. Outras diferenças podem ser estudadas [neste link](http://r4ds.had.co.nz/tibbles.html).

Vamos assumir que temos a seguinte base de dados:




```r
pnud_min
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### As cinco funções principais do `dplyr`

- `filter`
- `mutate`
- `select`
- `arrange`
- `summarise`

### Características

- O _input_  é sempre uma `tibble`, e o _output_  é sempre um `tibble`.
- No primeiro argumento colocamos o `tibble`, e nos outros argumentos colocamos o que queremos fazer.
- A utilização é facilitada com o emprego do operador `%>%`

### Vantagens

- Utiliza `C` e `C++` por trás da maioria das funções, o que geralmente torna o código mais eficiente.
- Pode trabalhar com diferentes fontes de dados, como bases relacionais (SQL) e `data.table`.

### `select`

- Utilizar `starts_with(x)`, `contains(x)`, `matches(x)`, `one_of(x)`, etc.
- Possível colocar nomes, índices, e intervalos de variáveis com `:`.


```r
pnud_min %>% 
  select(ano, regiao, muni)
## Error in eval(expr, envir, enclos): could not find function "%>%"
```


```r
pnud_min %>% 
  select(ano:regiao, rdpc)
## Error in eval(expr, envir, enclos): could not find function "%>%"
```


```r
pnud_min %>% 
  select(ano, starts_with('idhm'))
## Error in eval(expr, envir, enclos): could not find function "%>%"
```

### `filter`

- Parecido com `subset`.
- Condições separadas por vírgulas é o mesmo que separar por `&`.


```r
pnud_min %>% 
  select(ano, muni, uf) %>% 
  filter(uf == 'AC')
## Error in eval(expr, envir, enclos): could not find function "%>%"
```


```r
pnud_min %>% 
  select(ano, regiao, uf, idhm) %>% 
  filter(uf %in% c('SP', 'MG') | idhm > .5, ano == 2010)
## Error in eval(expr, envir, enclos): could not find function "%>%"
  # é igual a
  # filter(uf %in% c('SP', 'MG') | idhm > .5 & ano == 2010)
```



```r
library(stringr)
pnud_min %>% 
  select(muni, ano, uf) %>% 
  filter(str_detect(muni, '^[HG]|S$'), 
         ano == 1991)
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### `mutate`

- Parecido com `transform`, mas aceita várias novas colunas iterativamente.
- Novas variáveis devem ter o mesmo `length` que o `nrow` do bd oridinal ou `1`.


```r
pnud_min %>% 
  select(muni, rdpc, pop, idhm_l, espvida) %>% 
  mutate(renda = rdpc * pop, 
         razao = idhm_l / espvida)
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### `arrange`

- Simplesmente ordena de acordo com as opções.
- Utilizar `desc` para ordem decrescente.


```r
pnud_min %>% 
  filter(ano == 2010) %>% 
  arrange(desc(espvida))
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### `summarise`

- Retorna um vetor de tamanho `1` a partir de uma conta com as variáveis.
- Geralmente é utilizado em conjunto com `group_by`.
- Algumas funções importantes: `n()`, `n_distinct()`.


```r
pnud_min %>% 
  group_by(regiao, uf) %>% 
  summarise(n = n(), espvida = mean(espvida)) %>% 
  arrange(regiao, desc(espvida))
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```


```r
pnud_min %>% 
  filter(ano == 2010) %>% 
  count(regiao, sort = TRUE) %>% 
  mutate(prop = n / sum(n), prop = scales::percent(prop))
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### `gather`

- "Empilha" o banco de dados


```r
pnud_min %>% 
  select(uf, muni, ano, starts_with('idhm_')) %>% 
  gather(tipo_idhm, idhm, starts_with('idhm_')) %>% 
  arrange(desc(idhm))
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### `spread`

- "Joga" uma variável nas colunas
- É essencialmente a função inversa de `gather`


```r
pnud_min %>% 
  select(muni, uf, ano, starts_with('idhm_')) %>% 
  gather(tipo_idhm, idhm, starts_with('idhm_')) %>% 
  spread(ano, idhm)
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### Funções auxiliares

- `unite` junta duas ou mais colunas usando algum separador (`_`, por exemplo).
- `separate` faz o inverso de `unite`, e uma coluna em várias usando um separador.



```r
pnud_min %>% 
  select(muni, uf, ano, starts_with('idhm_')) %>% 
  gather(tipo_idhm, idhm, starts_with('idhm_')) %>% 
  separate(tipo_idhm, c('idhm_nm', 'tipo'), sep = '_') %>% 
  select(-idhm_nm) %>% 
  filter(ano == 2010) %>% 
  group_by(tipo) %>% 
  summarise(maior = muni[which.max(idhm)], idhm = max(idhm)) %>% 
  arrange(tipo, desc(idhm))
## Error in eval(expr, envir, enclos): object 'pnud_min' not found
```

### Um pouco mais de transformação de dados

- Para juntar tabelas, usar `inner_join`, `left_join`, `anti_join`, etc.
- Para realizar operações mais gerais, usar `do`.
- Para retirar duplicatas, utilizar `distinct`.







<script src="https://cdn.datacamp.com/datacamp-light-latest.min.js"></script>




<script src="https://cdn.datacamp.com/datacamp-light-latest.min.js"></script>



1. Calcule o número de ouro no R.

$$
\frac{1 + \sqrt{5}}{2}
$$

<div data-datacamp-exercise data-height="300" data-encoded="true">eyJsYW5ndWFnZSI6InIiLCJzYW1wbGUiOiIjIERpZ2l0ZSBhIGV4cHJlc3NcdTAwZTNvIHF1ZSBjYWxjdWxhIG8gblx1MDBmYW1lcm8gZGUgb3Vyby4iLCJzb2x1dGlvbiI6IigxICsgc3FydCg1KSkvMiIsInNjdCI6InRlc3Rfb3V0cHV0X2NvbnRhaW5zKFwiMS42MTgwMzRcIiwgaW5jb3JyZWN0X21zZyA9IFwiVGVtIGNlcnRlemEgZGUgcXVlIGluZGljb3UgYSBleHByZXNzXHUwMGUzbyBjb3JyZXRhbWVudGU/XCIpXG5zdWNjZXNzX21zZyhcIkNvcnJldG8hXCIpIn0=</div>






