# R, dplyr e SQL

# MySQL e dplyr

Uma das soluções mais simples para trabalharmos com bases de dados grandes em R é, em vez de carregar os dados na memória, conectar o R a uma fonte remota de dados e usar a gramática que aprendemos nos tutoriais anteriores para manipularmos os dados. Veremos a seguir um exemplo para dados em um servidor MySQL, mas ele vale para outros sistemas de gerenciamento de dados como PostgreSQL, MariaDB ou BigQuery. Vamos usar no tutorial uma adaptação do exemplo e dos dados do curso "Data Manipulation in R with dplyr", organizado por Gareth Grolemund, no site Datacamp.

O MySQL não armazena os dados na memória RAM, mas no disco rígido. Isso expande bastante o limite do tamanho dos dados que conseguiremos gerenciar, sem, no entanto, precisar de outra "gramática" para manipulação de dados.

```{r}
install.packages("RMySQL")
library(dplyr)
library(RMySQL)
```

O primeiro passo importante para trabalhar com dados em um servidor MySQL é fazer a conexão com uma base de dados. Vamos supor que temos um banco de dados (que, para o MySQL, significa um conjunto de tabelas, e não apenas uma tabela) chamado "PBF" em um servidor local, ou seja, no próprio computador, e que dentro desse banco de dados existe a tabela "transferencias201701". O usuário e senha fictícias são, respectivamente, "root" e "pass". Usamos, então, a função _src\_mysql_ para criar um objeto de "conexão", que chamaremos de "bd_mysql". Obviamente, o código abaixo não funcionará no seu computador, pois tal servidor não existe.

```{r}
conexao <- src_mysql(dbname = "PBF", 
                   user = "root",
                   password = "pass")
```

A seguir, criamos um objeto "tabela", que mantém a conexão direto com a tabela "transferencias201701" que está no banco de dados "PBF".

```{r}
tabela <- tbl(conexao, "pagamentos201701")
```

NOTA IMPORTANTE: nos testes que fizemos em sala de aula com uma conexão a um servidor PostgreSQL, tivemos que substituir o nome da tabela por um "statment" de SQL (veja [Issue 244 no repo do dplyr](https://github.com/tidyverse/dplyr/issues/244) para a solução). Vamos supor que nossa tabela tenha o nome "tabela" e esteja num schema denominado "schema". Para repetir o comando acima neste caso fazemos:

```{r}
tabela <- tbl(conexao, sql("SELECT * FROM schema.tabela"))
```

Simples, não? A partir deste ponto, basta trabalhar com objeto "tabela" da mesma maneira que trabalhamos _data frames_ até agora. A gramática oferecida pelo pacote _dplyr_ funciona normalmente e, quando quisermos, podemos exportar os dados com os métodos acima.

## Vôos em atraso - um exemplo do site Datacamp

Vamos ao exemplo do site Datacamp. Os códigos abaixo funcionarão normalmente no seu computador. O site Datacamp mantém um servidor remoto de MySQL que contém um banco de dados denominado "dplyr". Esta, por sua vez, contém uma tabela também chamada "dplyr" (um pouco confuso, pois não precisaria ter o mesmo nome da base de dados) com informações sobre vôos de companhias aéreas.

Comecemos estabelecendo a conexão com o servidor e o banco de dados, tal como fizemos anteriormente. A diferença, agora, é que estamos lidando com um servidor remoto e por isso precisamos informar seu "endereço". Fazemos isso preenchendo os argumentos "host" e "port", cujos valores foram fornecidos pelos criadores do servidor, além, obviamente, do nome do banco de dados, usuário e senha ("student" e "datacamp", respectivamente). 

```{r}
conexao <- src_mysql(dbname = "dplyr", 
                   host = "courses.csrrinzqubik.us-east-1.rds.amazonaws.com", 
                   port = 3306, 
                   user = "student",
                   password = "datacamp")
```

Para ver todas as tabelas disponíveis na base de dados com a qual fizemos a conexão usamos:

```{r}
src_tbls(conexao)
```

Uma vez conectados com o servidor, fazemos a conexão direta com a tabela "dplyr", que é a que nos interessa. Vamos chamar o objeto criado de "voos". Será este objeto que manipularemos com a gramática que aprendemos anteriormente. 

```{r}
voos <- tbl(conexao, "dplyr")
```

Você pode examinar o conteúdo de voos com as funções que já conhecemos: _head_, _str_, etc (Pela natureza do SQL, ele não nos retorna o número de observações, somente a de variáveis - mas isso é normal e você não precisa se preocupar com isso). Para o nosso exercício, vamos criar a variável "atraso", que é a soma as variáveis atrasos de partidas e chegadas de cada vôo, respectivamente "dep_delay" e "arr_delay". A seguir, vamos agrupar por companhia aérea (variável "carrier"), e calcular a soma agregada dos atrasos das companhias aéras. Tudo isso ser armazenado no objeto "voos_atraso". Note que não há diferença alguma em relação ao que fizemos no passado, exceto o fato de que estamos trabalhando com os dados em um servidor remoto.

```{r}
voos_atraso <- voos %>% 
  mutate(atraso =  dep_delay + arr_delay) %>%
  group_by(carrier) %>%
  summarise(total_atraso = sum(atraso))
```

## Tradução dplyr-SQL

Podemos traduzir, literalmente, o comando acima para a linguagem de R para SQL como a função _show\_query_

```{r}
show_query(voos_atraso)
```

O output, impresso no console, é a query equivalente que deveríamos produzir no MySQL para executar a mesma tarefa.

## Outras bases de dados

A conexão a outras bases de dados, MariaDb, SQLite, PostgreSQL e BigQuery, feita exatamente do mesmo modo que fizemos a conexão com um servidor de MySQL. Para as bases citadas usamos, respectivamente, as funções _scr\_mysql_ (sim, para MariaDB a função é a mesma que para MySQL), _scr\_sqlite_, _scr\_postgres_ e _scr\_bigquery_ com os mesmos 5 argumentos que vimos acima: dbname, host, port, user e password.

Os verbos de manipulação de dados do pacote dplyr funcionam normalmente para tabelas nesses servidores.

## Um exemplo com mais de uma tabela

Vamos seguir com outro exemplo de manipulação de dados num servidor MySQL (novamente no servidor do Datacamp) para trabalhar com mais de uma tabela e reforçar o que vimos acima.

Comecemos com a conexão:

```{r}
conexao <- src_mysql(dbname = "tweater", 
                   host = "courses.csrrinzqubik.us-east-1.rds.amazonaws.com", 
                   port = 3306, 
                   user = "student",
                   password = "datacamp")
```

_src\_tables_ fornece a listagem de tabelas na base de dados.

```{r}
src_tbls(conexao)
```

Nessa base de dados há 3 tabelas com informações provenientes de uma rede social: "users", que é uma tabela de usuários; "tweats" com informações sobre postagens de usuários; e "comments", com comentários às postagens por outros usuários.

Se você sabe algo de SQL, pode usar a função _tbl_ permite fazer _queries_ em linguagem SQL. Por exemplo.

```{r}
tbl(conexao, sql("SELECT * FROM comments WHERE user_id > 4"))
```

Mas queremos evitar o uso da linguagem SQL. Com _tbl_, criamos um objeto de tabela que seja manipulável com as funções do _dplyr_, sem, no entanto, importá-la. Vamos fazer isso para as três tabelas da base de dados que estamos usando:

```{r}
comments <- tbl(conexao, "comments")
tweats <- tbl(conexao, "tweats")
users <- tbl(conexao, "users")
```

Repetindo a _query_ acima, em que selecionamos na tabela _comments_ apenas as linhas com _user\_id_ > 4:

```{r}
filter(comments, user_id > 4)
```

A partir de agora, as funções de manipulação de dados do _dplyr_ são aplicáveis aos novos objetos criados para representar as tabelas que estão no servidor. Por exemplo, vamos renomear a variável "id" em tweats para "tweat_id" e fazer um _left join_ entre _comments_ e _tweats_ por "tweat_id":

```{r}
tweats2 <- rename(tweats, tweat_id = id)
tabela_join <- left_join(tweats2, comments, "tweat_id")
head(tabela_join)
```

Note que "tweats2" é uma tabela gerada no servidor de SQL e não está na memória RAM de nosso computador.

Novamente, podemos traduzir a query de R para SQL:

```{r}
show_query(tabela_join)
```

As funções da gramática do _dplyr_ -- _filter_, _select_, _rename_, _mutate_, _group\_by_, _summarize_, _left\_join_, _right\_join_, _inner\_join_, _full\_join_ e etc -- funcionam normalmente à tabelas remotas e facilitam demais a manipulação. Um exemplo tolo, porém completo, usando o operador %>% ( _pipe_, que permite omitir o primeiro argumento de cada função e nos poupa de repetir o nome da tabela diversas vezes) para ilustrar a aplicação de diversas funções do _dplyr_ ao mesmo tempo:

```{r}
tweats <- tweats %>% 
  rename(tweat_id = id) %>%              # renomeia variavel
  mutate(post = toupper(post)) %>%       # post em letras maiusculas
  left_join(comments, "tweat_id") %>%    # left join con tabela "comments"
  select(tweat_id, post, message) %>%    # mantem apenas 3 variaveis
  mutate(n_caract = nchar(message)) %>%  # cria var num caractecteres do comentario
  filter(n_caract < 10)                  # filtra por comentarios com menos de 10 caract

head(tweats, 7)
```

Uma maneira simples de trazer à memória de seu computador a tabela gerada a partir da query, com _as.data.frame_ importamos a tabela como _data frame_:

```{r}
tabela <- as.data.frame(tweats)
```

Note que "tabela", diferentemente de "tweats", é um _data frame_ no seu _workspace_.

## Tabelas temporárias _versus_ criação de tabelas no MySQL

Quando utilizamos os verbos do _dplyr_ para manipulação de dados em servidor MySQL, todas as consultas são geradas como tabelas temporárias no servidor. Como fazer com que as consultas se tornem tabelas permanentes no servidor?

Vamos trabalhar com um servidor fictício, pois não temos permissão para gerar tabelas no servidor que utilizamos como exemplo no tutorial. Vamos supor que temos uma tabela "pagamentos201701" na nossa base de dados "PBF" e que tal tabela contém uma variável "UF" para unidades da federação:

```{r}
conexao <- src_mysql(dbname = "PBF", 
                   user = "root",
                   password = "pass")
tabela <- tbl(conexao, "pagamentos201701")
minha_query <- tabela %>% filter(UF == "ES")
```

Ao produzir o comando acima, na prática, nada aconteceu. A execução da query só ocorrerá quando tentarmos trazer a tabela para a memória ("fetch") ou explicitarmos que ela deve ser computada.

Se quisermos trazer os dados para a memória, utilizamos a função _collect_.

```{r}
pagamentos_es <- collect(minha_query)
```

Ao usar o comando _collect_, a query é executada no servidor e os dados enviados ao R.

O caminho inverso -- subir ao servidor uma tabela -- é feito com a função _copy\_to_

```{r}
copy_to(dest = conexao, df = pagamentos_es, name = "pagamentos201701_es")
```

No entanto, _copy\_to_ não geram uma nova tabela no servidor. Para que uma nova tabela seja gerada, é preciso definir o argumento "temporary" como "FALSE" (o padrão é "TRUE"): 

```{r}
copy_to(dest = conexao, df = pagamentos_es, name = "pagamentos201701_es", temporary = FALSE)
```

Para executar a query no servidor sem que precisemos trazer a tabela e reenviá-la devemos usar a função _compute_, que também tem o argumento "temporary".

```{r}
compute(minha_query, name = "pagamentos201701_es", temporary = FALSE)
```

Sem definir "temporary" como "FALSE", a query será executada e a tabela gerada será temporária, apenas.
