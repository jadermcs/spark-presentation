o que é o spark

o paradigma map reduce, inspirado na programação funcional consiste de formulação de computações em duas etapas principais, o map e o reduce
seguido de outras etapas intermediarias, como shuffle e sort.
a operação map consiste em aplicar uma função unária a cada elemento de um conjunto de dados e o reduce trata da agregação desses dados por uma operação associativa.
é importante ressaltar que todo o processo é distribuido e assincrono, não há qualquer garantia de ordem entre os dados.
Na pratica cada computador trabalha com uma parte do conjunto de dados e se comunicam passando dados de um para o outro apenas para o reduce.
O HDFS parte do ecosistema hadoop, trata de fazer essa abstração da distribuição dos dados como também o shuffle e o sort,
 de tal forma que quando ser for trabalhar com um dado o caminho dele é apenas um link.
stack do spark prove camadas de abstração como o spark sql, streaming, mllib e graphx.



# Carregando as bibliotecas
library(sparklyr)

library(dplyr)

library(nycflights13)

library(ggplot2)

# Diferença entre ler disco e ler hdfs


# Conectando ao spark

sc <- spark_connect(master="yarn", limitar cores e memoria)
%sc armazena nossa conexão, precisando passa-la para as operações de leitura e escrita 
flights <- copy_to(sc, flights, "flights")
%copiamos os dados para a nossa conexão como o nome de "flights" esse é o nome no nosso datawarehouse temporario
airlines <- copy_to(sc, airlines, "airlines")

src_tbls(sc)
%lista tabelas temporarias
# equivalencia SQL

% o sparklyr traduz para uma query HIVEQL com as seguintes equivalencias

select ~ SELECT

filter ~ WHERE

arrange ~ ORDER

summarise ~ aggregators: sum, min, sd, etc.

mutate ~ operators: +, *, log, etc.

% existem outras funções especiais do hiveql que vai alem do escopo desse curso mas podem ser facilmente
acessadas no site do hive.

# workaround

select(flights, year:day, arr_delay, dep_delay)

filter(flights, dep_delay > 1000)

arrange(flights, desc(dep_delay))

c4 <- flights %>%
  filter(month == 5, day == 17, carrier %in% c('UA', 'WN', 'AA', 'DL')) %>%
  select(carrier, dep_delay, air_time, distance) %>%
  arrange(carrier) %>%
  mutate(air_time_hours = air_time / 60)

c4 %>%
  group_by(carrier) %>%
  summarize(count = n(), mean_dep_delay = mean(dep_delay))

sample_frac(flights, 0.01)

carrierhours <- collect(c4)

sdf_persist(x, storage.level = "MEMORY_AND_DISK")

tbl <- spark_read_parquet(sc, "data", "hdfs://hdfs.company.org:9000/hdfs-path/data")

# Hive function
flights %>%
  mutate(flight_date = paste(year,month,day,sep="-"),
         days_since = datediff(current_date(), flight_date)) %>%
  group_by(flight_date,days_since) %>%
  tally() %>%
  arrange(-days_since)