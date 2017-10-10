---
title: 'Manipulação de dados'
date: '2017-10-10'
---





> "(...) The fact that data science exists as a field is a colossal failure of statistics. To me, what I do is what statistics is all about. It is gaining insight from data using modelling and visualization. Data munging and manipulation is hard and statistics has just said that’s not our domain."
> 
> Hadley Wickham

Esta seção trata do tema *transformação de dados*. Trata-se de uma tarefa dolorosa e demorada, tomando muitas vezes a maior parte do tempo de uma análise estatística. Essa etapa é essencial em qualquer análise de dados e, apesar de negligenciada pela academia, é decisiva para o sucesso de estudos aplicados.

Usualmente, o cientista de dados parte de uma base "crua" e a transforma até obter uma base de dados analítica. A base crua pode ser não estruturada, semi-estruturada ou estruturada. Já a base analítica é necessariamente estruturada e está preparada para passar por análises estatísticas a menos de transformações simples. 

Um conceito importante para obtenção de uma base analítica é o *data tidying*, ou arrumação de dados. Uma base é considerada *tidy* se

1. Cada linha da base representa uma observação.
2. Cada coluna da base representa uma variável.
3. ~~Cada tabela considera informações de uma unidade amostral.~~

A base de dados analítica é estruturada de tal forma que pode ser colocada diretamente em sistemas de modelagem estatística ou de visualização. Nem sempre uma base de dados analítica está no formato *tidy*, mas usualmente são necessários poucos passos para migrar de uma para outra.

A Figura abaixo mostra a fase de "disputa" com os dados (*data wrangling*) para deixá-los no formato analítico.

<img src="http://r4ds.had.co.nz/diagrams/data-science-wrangle.png" title="Transformação no ciclo da ciência de dados." alt="Transformação no ciclo da ciência de dados." width="60%" height="60%" />

## Pacotes `dplyr` e `tidyr`

O `dplyr` é o pacote mais útil para realizar transformação de dados, aliando simplicidade e eficiência de uma forma elegante. Os scripts em `R` que fazem uso inteligente dos verbos `dplyr` e as facilidades do operador _pipe_ tendem a ficar mais legíveis e organizados sem perder velocidade de execução.

Antes de apresentar as principais funções do `dplyr` e do `tidyr`, precisamos trabalhar com o conceito de `tibbles`.

### Trabalhando com `tibble`'s

Uma `tibble` nada mais é do que um `data.frame`, mas com um método de impressão mais adequado. Outras diferenças podem ser estudadas [neste link](http://r4ds.had.co.nz/tibbles.html).

Considere a seguinte base de dados.




```r
pnud_min
## # A tibble: 16,686 x 14
##      ano                  muni    uf regiao  idhm idhm_e idhm_l idhm_r
##    <int>                 <chr> <chr>  <chr> <dbl>  <dbl>  <dbl>  <dbl>
##  1  1991 ALTA FLORESTA D'OESTE    RO  Norte 0.329  0.112  0.617  0.516
##  2  1991             ARIQUEMES    RO  Norte 0.432  0.199  0.684  0.593
##  3  1991                CABIXI    RO  Norte 0.309  0.108  0.636  0.430
##  4  1991                CACOAL    RO  Norte 0.407  0.171  0.667  0.593
##  5  1991            CEREJEIRAS    RO  Norte 0.386  0.167  0.629  0.547
##  6  1991     COLORADO DO OESTE    RO  Norte 0.376  0.151  0.658  0.536
##  7  1991            CORUMBIARA    RO  Norte 0.203  0.039  0.572  0.373
##  8  1991         COSTA MARQUES    RO  Norte 0.425  0.220  0.629  0.553
##  9  1991       ESPIGÃO D'OESTE    RO  Norte 0.388  0.159  0.653  0.561
## 10  1991         GUAJARÁ-MIRIM    RO  Norte 0.468  0.247  0.662  0.625
## # ... with 16,676 more rows, and 6 more variables: espvida <dbl>,
## #   rdpc <dbl>, gini <dbl>, pop <int>, lat <dbl>, lon <dbl>
```

## Base IDH-Municipal - PNUD

Nessa seção, vamos trabalhar com uma base simplificada do [PNUD (Programa das Nações Unidas para o Desenvolvimento)](http://www.atlasbrasil.org.br/2013/pt/download/base/), contendo informações socioeconômicas de todos os municípios do país. Os resultados foram obtidos a partir dos Censos de 1991, 2000 e 2010.

A base contém 16686 linhas e 14 colunas, descritas abaixo:

- `ano` - Ano do Censo utilizado como base para cálculo do IDH-Municipal e outras métricas.
- `muni` - Nome do município. Cada município aparece três vezes, um para cada ano.
- `uf` - Unidade Federativa.
- `regiao` - Região brasileira.
- `idhm` - IDH municipal, dividido em
    - `idhm_e` - IDH municipal - educação.
    - `idhm_l` - IDH municipal - longevidade.
    - `idhm_r` - IDH municipal - renda.
- `espvida` - Expectativa de vida.
- `rdpc` - Renda *per capita*.
- `gini` - Coeficiente de gini municipal (mede desigualdade social).
- `pop` - População residente do município.
- `lat`, `lon` - Latitude e longitude do município (ponto médio).

## Os cinco verbos do `dplyr`

As funções principais do `dplyr` são:

- `filter` - filtra linhas
- `select` - seleciona colunas
- `mutate` - cria/modifica colunas
- `arrange` - ordena a base
- `summarise` - sumariza a base

### Características

- O _input_  é sempre uma `tibble` e o _output_  é sempre um `tibble`.
- Colocamos o `tibble` no primeiro argumento e o que queremos fazer nos outros argumentos.
- A utilização é facilitada com o emprego do operador `%>%`.
- O pacote faz uso extensivo de NSE (*non standard evaluation*).

### Vantagens

- Utiliza `C` e `C++` por trás da maioria das funções, o que geralmente torna o código mais eficiente.
- Pode trabalhar com diferentes fontes de dados, como bases relacionais (SQL) e `data.table`.

-----------------------------------------------------

## `select`

- Utilizar as funções `starts_with(x)`, `contains(x)`, `matches(x)`, `one_of(x)`.
- Possível colocar nomes, índices e intervalos de variáveis com `:`.


```r
pnud_min %>% 
  select(ano, regiao, muni)
## # A tibble: 16,686 x 3
##      ano regiao                  muni
##    <int>  <chr>                 <chr>
##  1  1991  Norte ALTA FLORESTA D'OESTE
##  2  1991  Norte             ARIQUEMES
##  3  1991  Norte                CABIXI
##  4  1991  Norte                CACOAL
##  5  1991  Norte            CEREJEIRAS
##  6  1991  Norte     COLORADO DO OESTE
##  7  1991  Norte            CORUMBIARA
##  8  1991  Norte         COSTA MARQUES
##  9  1991  Norte       ESPIGÃO D'OESTE
## 10  1991  Norte         GUAJARÁ-MIRIM
## # ... with 16,676 more rows
```


```r
pnud_min %>% 
  select(ano:regiao, rdpc)
## # A tibble: 16,686 x 5
##      ano                  muni    uf regiao   rdpc
##    <int>                 <chr> <chr>  <chr>  <dbl>
##  1  1991 ALTA FLORESTA D'OESTE    RO  Norte 198.46
##  2  1991             ARIQUEMES    RO  Norte 319.47
##  3  1991                CABIXI    RO  Norte 116.38
##  4  1991                CACOAL    RO  Norte 320.24
##  5  1991            CEREJEIRAS    RO  Norte 240.10
##  6  1991     COLORADO DO OESTE    RO  Norte 224.82
##  7  1991            CORUMBIARA    RO  Norte  81.38
##  8  1991         COSTA MARQUES    RO  Norte 250.08
##  9  1991       ESPIGÃO D'OESTE    RO  Norte 263.03
## 10  1991         GUAJARÁ-MIRIM    RO  Norte 391.37
## # ... with 16,676 more rows
```


```r
pnud_min %>% 
  select(ano, starts_with('idhm'))
## # A tibble: 16,686 x 5
##      ano  idhm idhm_e idhm_l idhm_r
##    <int> <dbl>  <dbl>  <dbl>  <dbl>
##  1  1991 0.329  0.112  0.617  0.516
##  2  1991 0.432  0.199  0.684  0.593
##  3  1991 0.309  0.108  0.636  0.430
##  4  1991 0.407  0.171  0.667  0.593
##  5  1991 0.386  0.167  0.629  0.547
##  6  1991 0.376  0.151  0.658  0.536
##  7  1991 0.203  0.039  0.572  0.373
##  8  1991 0.425  0.220  0.629  0.553
##  9  1991 0.388  0.159  0.653  0.561
## 10  1991 0.468  0.247  0.662  0.625
## # ... with 16,676 more rows
```

-----------------------------------------------------

## `filter`

- Parecido com `subset`.
- Condições separadas por vírgulas equivalem a separar por `&`.


```r
pnud_min %>% 
  select(ano, muni, uf) %>% 
  filter(uf == 'AC')
## # A tibble: 66 x 3
##      ano            muni    uf
##    <int>           <chr> <chr>
##  1  1991      ACRELÂNDIA    AC
##  2  1991    ASSIS BRASIL    AC
##  3  1991       BRASILÉIA    AC
##  4  1991          BUJARI    AC
##  5  1991        CAPIXABA    AC
##  6  1991 CRUZEIRO DO SUL    AC
##  7  1991  EPITACIOLÂNDIA    AC
##  8  1991           FEIJÓ    AC
##  9  1991          JORDÃO    AC
## 10  1991     MÂNCIO LIMA    AC
## # ... with 56 more rows
```

Para fazer várias condições, use os operadores lógicos `&` e `|` ou separe filtros entre vírgulas.

<div class='admonition note'>
<p class='admonition-title'>
%in%
</p>
<p>
<b>%in%</b> é um operador muito útil para trabalhar com vetores. O resultado da operação é um vetor lógico do tamanho do vetor do elemento da esquerda, identificando quais elementos da esquerda batem com algum elemento da direita.
</p>
</div>


```r
pnud_min %>% 
  select(ano, regiao, uf, idhm) %>% 
  filter(uf %in% c('SP', 'MG') | idhm > .5, ano == 2010)
## # A tibble: 5,527 x 4
##      ano regiao    uf  idhm
##    <int>  <chr> <chr> <dbl>
##  1  2010  Norte    RO 0.641
##  2  2010  Norte    RO 0.702
##  3  2010  Norte    RO 0.650
##  4  2010  Norte    RO 0.718
##  5  2010  Norte    RO 0.692
##  6  2010  Norte    RO 0.685
##  7  2010  Norte    RO 0.613
##  8  2010  Norte    RO 0.611
##  9  2010  Norte    RO 0.672
## 10  2010  Norte    RO 0.657
## # ... with 5,517 more rows
  # é igual a
  # filter(uf %in% c('SP', 'MG') | idhm > .5 & ano == 2010)
```


```r
library(stringr)
pnud_min %>% 
  select(muni, ano, uf) %>% 
  filter(str_detect(muni, '^[HG]|S$'), 
         ano == 1991)
## # A tibble: 970 x 3
##                         muni   ano    uf
##                        <chr> <int> <chr>
##  1                 ARIQUEMES  1991    RO
##  2                CEREJEIRAS  1991    RO
##  3             COSTA MARQUES  1991    RO
##  4             GUAJARÁ-MIRIM  1991    RO
##  5   ALTO ALEGRE DOS PARECIS  1991    RO
##  6                   BURITIS  1991    RO
##  7              CASTANHEIRAS  1991    RO
##  8 GOVERNADOR JORGE TEIXEIRA  1991    RO
##  9                   PARECIS  1991    RO
## 10              SERINGUEIRAS  1991    RO
## # ... with 960 more rows
```

-----------------------------------------------------

## `mutate`

- Parecido com `transform`, mas aceita várias novas colunas iterativamente.
- Novas variáveis devem ter o mesmo `length` que o `nrow` do bd oridinal ou `1`.


```r
pnud_min %>% 
  select(muni, rdpc, pop, idhm_l, espvida) %>% 
  mutate(renda = rdpc * pop, 
         razao = idhm_l / espvida)
## # A tibble: 16,686 x 7
##                     muni   rdpc   pop idhm_l espvida      renda
##                    <chr>  <dbl> <int>  <dbl>   <dbl>      <dbl>
##  1 ALTA FLORESTA D'OESTE 198.46 22835  0.617   62.01  4531834.1
##  2             ARIQUEMES 319.47 55018  0.684   66.02 17576600.5
##  3                CABIXI 116.38  5846  0.636   63.16   680357.5
##  4                CACOAL 320.24 66534  0.667   65.03 21306848.2
##  5            CEREJEIRAS 240.10 19030  0.629   62.73  4569103.0
##  6     COLORADO DO OESTE 224.82 25070  0.658   64.46  5636237.4
##  7            CORUMBIARA  81.38 10737  0.572   59.32   873777.1
##  8         COSTA MARQUES 250.08  6902  0.629   62.76  1726052.2
##  9       ESPIGÃO D'OESTE 263.03 22505  0.653   64.18  5919490.1
## 10         GUAJARÁ-MIRIM 391.37 31240  0.662   64.71 12226398.8
## # ... with 16,676 more rows, and 1 more variables: razao <dbl>
```

-----------------------------------------------------

## `arrange`

- Simplesmente ordena de acordo com as opções.
- Utilizar `desc` para ordem decrescente.


```r
pnud_min %>% 
  filter(ano == 2010) %>% 
  arrange(desc(espvida))
## # A tibble: 5,562 x 14
##      ano               muni    uf regiao  idhm idhm_e idhm_l idhm_r
##    <int>              <chr> <chr>  <chr> <dbl>  <dbl>  <dbl>  <dbl>
##  1  2010           BLUMENAU    SC    Sul 0.806  0.722  0.894  0.812
##  2  2010            BRUSQUE    SC    Sul 0.795  0.707  0.894  0.794
##  3  2010 BALNEÁRIO CAMBORIÚ    SC    Sul 0.845  0.789  0.894  0.854
##  4  2010         RIO DO SUL    SC    Sul 0.802  0.727  0.894  0.793
##  5  2010    RANCHO QUEIMADO    SC    Sul 0.753  0.644  0.893  0.743
##  6  2010       RIO DO OESTE    SC    Sul 0.754  0.625  0.892  0.769
##  7  2010             IOMERÊ    SC    Sul 0.795  0.749  0.891  0.754
##  8  2010            JOAÇABA    SC    Sul 0.827  0.771  0.891  0.823
##  9  2010        NOVA TRENTO    SC    Sul 0.748  0.628  0.891  0.749
## 10  2010        PORTO UNIÃO    SC    Sul 0.786  0.724  0.891  0.752
## # ... with 5,552 more rows, and 6 more variables: espvida <dbl>,
## #   rdpc <dbl>, gini <dbl>, pop <int>, lat <dbl>, lon <dbl>
```

-----------------------------------------------------

## `summarise`

- Retorna um vetor de tamanho `1` a partir de uma conta com as variáveis.
- Geralmente é utilizado em conjunto com `group_by`.
- Algumas funções importantes: `n()`, `n_distinct()`.


```r
pnud_min %>% 
  group_by(regiao, uf) %>% 
  summarise(n = n(), espvida = mean(espvida)) %>% 
  arrange(regiao, desc(espvida))
## # A tibble: 27 x 4
## # Groups:   regiao [5]
##          regiao    uf     n  espvida
##           <chr> <chr> <int>    <dbl>
##  1 Centro-Oeste    DF     3 73.36000
##  2 Centro-Oeste    GO   735 69.95346
##  3 Centro-Oeste    MS   234 69.94291
##  4 Centro-Oeste    MT   423 69.42915
##  5     Nordeste    CE   552 65.60627
##  6     Nordeste    RN   501 65.11439
##  7     Nordeste    PE   555 64.92721
##  8     Nordeste    BA  1251 64.62361
##  9     Nordeste    SE   225 64.26031
## 10     Nordeste    PI   672 64.04028
## # ... with 17 more rows
```


```r
pnud_min %>% 
  filter(ano == 2010) %>% 
  count(regiao, sort = TRUE) %>% 
  mutate(prop = n / sum(n), prop = scales::percent(prop))
## # A tibble: 5 x 3
##         regiao     n  prop
##          <chr> <int> <chr>
## 1     Nordeste  1794 32.3%
## 2      Sudeste  1667 30.0%
## 3          Sul  1187 21.3%
## 4 Centro-Oeste   465  8.4%
## 5        Norte   449  8.1%
```


-----------------------------------------------------

## `gather`

- "Empilha" o banco de dados


```r
pnud_min %>% 
  select(uf, muni, ano, starts_with('idhm_')) %>% 
  gather(tipo_idhm, idhm, starts_with('idhm_')) %>% 
  arrange(desc(idhm))
## # A tibble: 50,058 x 5
##       uf               muni   ano tipo_idhm  idhm
##    <chr>              <chr> <int>     <chr> <dbl>
##  1    SC BALNEÁRIO CAMBORIÚ  2010    idhm_l 0.894
##  2    SC           BLUMENAU  2010    idhm_l 0.894
##  3    SC            BRUSQUE  2010    idhm_l 0.894
##  4    SC         RIO DO SUL  2010    idhm_l 0.894
##  5    SC    RANCHO QUEIMADO  2010    idhm_l 0.893
##  6    SC       RIO DO OESTE  2010    idhm_l 0.892
##  7    SC             IOMERÊ  2010    idhm_l 0.891
##  8    SC            JOAÇABA  2010    idhm_l 0.891
##  9    SC        NOVA TRENTO  2010    idhm_l 0.891
## 10    SC        PORTO UNIÃO  2010    idhm_l 0.891
## # ... with 50,048 more rows
```

-----------------------------------------------------

## `spread`

- "Joga" uma variável nas colunas
- É essencialmente a função inversa de `gather`


```r
pnud_min %>% 
  select(muni, uf, ano, starts_with('idhm_')) %>% 
  gather(tipo_idhm, idhm, starts_with('idhm_')) %>% 
  spread(ano, idhm)
## # A tibble: 16,686 x 6
##                   muni    uf tipo_idhm `1991` `2000` `2010`
##  *               <chr> <chr>     <chr>  <dbl>  <dbl>  <dbl>
##  1     ABADIA DE GOIÁS    GO    idhm_e  0.183  0.386  0.622
##  2     ABADIA DE GOIÁS    GO    idhm_l  0.658  0.765  0.830
##  3     ABADIA DE GOIÁS    GO    idhm_r  0.563  0.623  0.687
##  4 Abadia dos Dourados    MG    idhm_e  0.225  0.387  0.563
##  5 Abadia dos Dourados    MG    idhm_l  0.728  0.799  0.839
##  6 Abadia dos Dourados    MG    idhm_r  0.551  0.616  0.693
##  7           ABADIÂNIA    GO    idhm_e  0.188  0.292  0.579
##  8           ABADIÂNIA    GO    idhm_l  0.656  0.730  0.841
##  9           ABADIÂNIA    GO    idhm_r  0.560  0.598  0.671
## 10              Abaeté    MG    idhm_e  0.180  0.385  0.556
## # ... with 16,676 more rows
```

-----------------------------------------------------

## Funções auxiliares

- `unite` junta duas ou mais colunas usando algum separador (`_`, por exemplo).
- `separate` faz o inverso de `unite`, transforma uma coluna em várias usando um separador.


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
## # A tibble: 3 x 3
##    tipo              maior  idhm
##   <chr>              <chr> <dbl>
## 1     e ÁGUAS DE SÃO PEDRO 0.825
## 2     l BALNEÁRIO CAMBORIÚ 0.894
## 3     r SÃO CAETANO DO SUL 0.891
```

-----------------------------------------------------

## Um pouco mais de transformação de dados

- Para juntar tabelas, usar `inner_join`, `left_join`, `anti_join` etc.
- Para realizar operações mais gerais, usar `do`.
- Para retirar duplicatas, utilizar `distinct`.






<script src="https://cdn.datacamp.com/datacamp-light-latest.min.js"></script>




