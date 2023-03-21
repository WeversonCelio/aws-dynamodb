# Desafio AWS Dynamodb

Os comandos usados para criação do desafio

## criar tabela Music

    aws dynamodb create-table \
        --table-name Music \
        --attribute-definitions \
            AttributeName=Artist,AttributeType=S \
            AttributeName=SongTitle,AttributeType=S \
        --key-schema \
            AttributeName=Artist,KeyType=HASH \
            AttributeName=SongTitle,KeyType=RANGE \
        --provisioned-throughput \
            ReadCapacityUnits=10,WriteCapacityUnits=5

* table-name criada a tabela musica
* attribute-definitions: Existem dois atributos, Artist e SongTitle do tipo String
* key-schema: Artist sera a partition key e a SongTitle sera a sort key
* A capacidade será d 10 units para leitura e 5 units para escrita

## Adicionando um item

    aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \

    * Foi adicionado o item do arquivo na tabela Music

## Adicionando items em lotes

    aws dynamodb batch-write-item \
        --request-items file://batchmusic.json

## Realizado pesquisas

* retornar musicas do artista "Jao"

        aws dynamodb query \
            --table-name Music \
            --key-condition-expression "Artist = :artist" \
            --expression-attribute-values '{":artist":{"S":"Jao"}}'

* nao e possivel retornar a pesquisa pois a partion key nao esta presente

        aws dynamodb query \
                --table-name Music \
                --key-condition-expression "SongTitle = :title" \
                --expression-attribute-values '{":title":{"S":"Coringa"}}'

## Criando item secundario

    aws dynamodb update-table \
        --table-name Music \
        --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
        --global-secondary-index-updates \
            "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
            \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

* o item secundario e baseado em album

        aws dynamodb query \
                --table-name Music \
                --index-name AlbumTitle-index \
                --key-condition-expression "AlbumTitle = :name" \
                --expression-attribute-values '{":name":{"S":"Pirata"}}'

* pesquisar pelo album Pirata

## criando item secundario com base no artista e album

    aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

* pesquisa com base neste secundario

        aws dynamodb query \
            --table-name Music \
            --index-name ArtistAlbumTitle-index \
            --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
            --expression-attribute-values '{":v_artist":{"S":"Jao"}, ":v_title":{"S":"Lobos"}}'

## criando item secundario com base no artista e ano

    aws dynamodb update-table \
        --table-name Music \
        --attribute-definitions\
            AttributeName=SongTitle,AttributeType=S \
            AttributeName=SongYear,AttributeType=S \
        --global-secondary-index-updates \
            "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
            \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

* pesquisa com base neste secundario

        aws dynamodb query \
            --table-name Music \
            --index-name SongTitleYear-index \
            --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
            --expression-attribute-values '{":v_song":{"S":"A Última Noite"}, ":v_year":{"S":"2019"}}'
