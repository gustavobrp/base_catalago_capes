# Base de dados com informações do Catálago de teses e dissertações da CAPES

Pelo link disponibilizado nessa página é possível baixar uma base de dados com as informações de teses e dissertações disponibilzadas no site de Dados Abertos da CAPES. A base contém informações das produções de mestrado e doutorado do período de **1987 a 2021**. As bases contam apenas com dados dos mestrados e doutorados acadêmicos e contém as seguintes informações: 

Para baixar essas bases eu utilizei como referência o [código em Python](https://github.com/meirelesff/catalogo_capes) disponibilizado por Fernando Meirelles, do IESP/UERJ, adaptando-o para o R. O código do download está disponibilizado abaixo. Após o download das bases, elas foram reúnidas em uma única base com as seguintes colunas: 

````r
 [1] "ano"               "codigo_programa"   "regiao"            "uf"                "sigla_ies"         "nome_ies"         
 [7] "nome_programa"     "grande_area"       "area_conhecimento" "area_avaliacao"    "autor"             "orientador"       
[13] "titulo"            "nivel"             "data_defesa"       "palavras_chave"    "resumo"            "url_sucupira"
````

A única modificação realizada sobre as bases foi o filtro para produções de mestrado e de doutorado, bem como a compatibilização dos nomes das colunas, a partir do código do Fernando. Portanto, análises a partir dessa base devem considerar trabalhos de limpeza e checagem das informações. A informação link do trabalho na Plataforma Sucupira (url_sucupira) existe apenas a partir de 2012.

Se houver problemas para baixar a base, bem como dúvidas ou sugestões, entre em contato.

**[Link para download da base](https://1drv.ms/u/s!AvSWV7-ZbuGLj7QMQJvO9QipKg3tOg?e=IoZrqt)** 

## Código para baixar as bases no R

Uma observação sobre o código: como durante o processo de download das bases o site da CAPES pode não responder, o código pode ter que ser executado novamente do início. Por essa razão, com ajuda do ChatGPT, eu incluí uma parte para que refizesse o download, mas checando se as bases que já tinham sido baixadas em execuções anteriores. Nada elegante, mas foi o que pensei para o momento ;-)

```r
library(rvest)
library(stringr)

urls <- c(
  'https://dadosabertos.capes.gov.br/dataset/1987-a-2012-catalogo-de-teses-e-dissertacoes-brasil',
  'https://dadosabertos.capes.gov.br/dataset/catalogo-de-teses-e-dissertacoes-de-2013-a-2016',
  'https://dadosabertos.capes.gov.br/dataset/2017-2020-catalogo-de-teses-e-dissertacoes-da-capes',
  'https://dadosabertos.capes.gov.br/dataset/2021-a-2024-catalogo-de-teses-e-dissertacoes-brasil'
)

# lista para armazenar os links
todos_links <- list()

for (url in urls) {
  # scraping da página
  pagina <- tryCatch(read_html(url), error = function(e) NULL)
  
  if (!is.null(pagina)) {
    links_pagina <- pagina %>% 
      html_nodes("a") %>% 
      html_attr("href")
    
    todos_links[[length(todos_links) + 1]] <- links_pagina
  } else {
    cat("Não foi possível ler a página:", url, "\n") # aviso para caso dê problema para ler a página
  }
}

# combinando os links
links <- unlist(todos_links)

links_csv <- links[str_detect(links, ".csv") & str_detect(links, "\\d{4}")]

download_with_retry <- function(url, dest_file, max_attempts = 3, timeout = 10) {
  attempt <- 1
  while (attempt <= max_attempts) {
    tryCatch({
      download.file(url, dest_file, timeout = timeout)
      cat("Arquivo", dest_file, "baixado com sucesso.\n") # aviso para as bases baixadas
      break
    }, error = function(e) {
      cat("Erro ao baixar o arquivo", dest_file, "na tentativa", attempt, ":", conditionMessage(e), "\n") # aviso para caso haja erro no download. Pois o site da CAPES pode cair durante o download
      if (attempt < max_attempts) {
        cat("Tentando novamente...\n")
        Sys.sleep(5)  # Aguarda 5 segundos antes da próxima tentativa
      } else {
        cat("Limite máximo de tentativas atingido. Abortando...\n")
      }
    })
    attempt <- attempt + 1
  }
}

# checar se os arquivos já existem na pasta de trabalho
files_in_folder <- list.files(pattern = ".csv$")

# restringindo o download apenas das bases que não estão na pasta
for (link_csv in links_csv) {
  if (str_detect(link_csv, ".csv")) {
    filename <- basename(link_csv)
    if (!(filename %in% files_in_folder)) {
      download_with_retry(link_csv, filename)
    } else {
      cat("O arquivo", filename, "já existe na pasta de trabalho.\n")
    }
  }
}

```
