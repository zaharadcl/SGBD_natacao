# SGBD_natacao
1. Estratégia de Normalização (3ª Forma Normal)
A principal decisão de design foi a decomposição de atributos de texto em tabelas de domínio (Lookup Tables).

Entidades de Domínio (SEXO, ESTADO, ESTILO, CATEGORIA): * Decisão: Em vez de armazenar "Masculino" ou "Pernambuco" como strings repetidas nas tabelas de ATLETA ou CLUBE, isolamos esses dados.

Justificativa: Isso elimina a redundância e impede "anomalias de inserção". Garante que um estilo de nado seja sempre escrito da mesma forma, facilitando agrupamentos e filtros em relatórios de desempenho.

Integridade: O uso de UK (Unique Key) nessas tabelas garante que não existam duplicatas lógicas (ex: duas entradas para o mesmo estado).

2. Hierarquia de Localização e Filiação
ESTADO (UF) -> CLUBE -> ATLETA:

Decisão: Criamos uma hierarquia clara de dependência.

Justificativa: Um clube está sediado em um estado, e um atleta é filiado a um clube. Ao separar o ESTADO do CLUBE, permitimos análises macro (ex: ranking de medalhas por estado) sem precisar varrer e limpar strings de texto nos registros dos clubes.

3. Especialização da PROVA
Composição da PROVA:

Decisão: A entidade PROVA não armazena o nome do estilo ou a categoria diretamente; ela é uma tabela de interseção técnica que une distancia, id_estilo, id_sexo e id_categoria.

Justificativa: Isso permite que o sistema gerencie o calendário de eventos de forma dinâmica. Se a federação criar uma nova categoria (ex: "Master"), basta adicionar um registro na tabela CATEGORIA e vinculá-lo a novas PROVAS, sem alterar o código do sistema.

4. Decomposição de Performance (RESULTADO e PARCIAL)
Esta é a área mais crítica do banco, onde os dados de alta precisão são armazenados.

RESULTADO (Entidade de Agregação):

Decisão: Armazena o desfecho final do atleta em uma prova (tempo total, colocação e pontos).

Justificativa: Centraliza os dados necessários para o Quadro de Medalhas e Pontuação de Eficiência visíveis na interface.

PARCIAL (Entidade de Granularidade):

Decisão: Criada como uma entidade dependente (1:N) de RESULTADO.

Justificativa: Conforme solicitado, para suportar a soma de tempos que gera o total, a tabela PARCIAL permite armazenar múltiplos registros para um único resultado (ex: parciais de 50m em uma prova de 400m).

Atributo Ordem: O campo ordem é vital para garantir que a sequência dos nados seja respeitada no cálculo da soma e na exibição do histórico de ritmo do atleta.

5. Integridade Referencial e Tipagem
Relacionamentos Identificadores (--): * Utilizados entre ATLETA/PROVA -> RESULTADO e RESULTADO -> PARCIAL. Se um atleta for removido ou uma prova cancelada, os resultados e parciais vinculados perdem o sentido e devem sofrer exclusão lógica ou física (Cascade).

Relacionamentos Não Identificadores (..): * Utilizados para características biográficas ou técnicas (Sexo, Categoria, Estilo). O objeto principal (Atleta ou Prova) existe independentemente de uma alteração na descrição da tabela de domínio.

Tipos de Dados: * Uso de TIME para precisão de milésimos, DECIMAL para pontuações (evitando erros de arredondamento de ponto flutuante) e INT para chaves primárias visando performance em indexação.
````mermaid
erDiagram
    ESTADO ||--o{ CLUBE : "sedia"
    CLUBE ||..o{ ATLETA : "filia"
    SEXO ||..o{ ATLETA : "define"
    SEXO ||..o{ PROVA : "restringe"
    CATEGORIA ||..o{ ATLETA : "classifica"
    CATEGORIA ||..o{ PROVA : "agrupa"
    ESTILO ||..o{ PROVA : "caracteriza"
    ATLETA ||--o{ RESULTADO : "disputa"
    PROVA ||--o{ RESULTADO : "gera"
    RESULTADO ||--|{ PARCIAL : "decompõe"

    ESTADO {
        int id_estado PK
        string sigla UK
        string nome_estado
    }

    CLUBE {
        int id_clube PK
        string nome_clube
        int id_estado FK
    }

    SEXO {
        int id_sexo PK
        string descricao UK
    }

    ESTILO {
        int id_estilo PK
        string nome_estilo UK
    }

    CATEGORIA {
        int id_categoria PK
        string nome_categoria
    }

    ATLETA {
        int id_atleta PK
        string nome_atleta
        int id_sexo FK
        int id_clube FK
        int id_categoria FK
    }

    PROVA {
        int id_prova PK
        int distancia
        int id_estilo FK
        int id_sexo FK
        int id_categoria FK
    }

    RESULTADO {
        int id_resultado PK
        int id_atleta FK
        int id_prova FK
        int serie "Número da bateria/série do atleta"
        int raia "Número da raia ocupada (1 a 8)"
        time tempo_final
        int colocacao
        decimal pontos
    }

    PARCIAL {
        int id_parcial PK
        int id_resultado FK
        int ordem "Ex: 1 para os primeiros 50m"
        time tempo_parcial "Tempo gasto no trecho"
    }
````
  
