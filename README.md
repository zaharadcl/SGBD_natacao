# SGBD_natacao
1. Normalização de Domínios (Tabelas de Referência)
SEXO, ESTADO, ESTILO e CATEGORIA: Foram isoladas em tabelas próprias para cumprir a 3ª Forma Normal. Isso evita a "anomalia de atualização" (ter que mudar o nome de um estilo em mil provas diferentes) e garante a integridade dos dados, impedindo valores inválidos ou variações de escrita (ex: "Masc" vs "Masculino").

2. Relacionamentos Identificadores vs. Não Identificadores
Linhas Tracejadas (..): Utilizadas em CLUBE, SEXO, CATEGORIA e ESTILO. São relacionamentos não identificadores, pois um ATLETA ou uma PROVA possuem sua própria identidade (PK) e apenas referenciam essas tabelas como atributos estrangeiros (FK). Se o clube deixar de existir, o registro do atleta ainda possui integridade própria.

Linhas Sólidas (--): Utilizadas entre ATLETA/PROVA e RESULTADO. O RESULTADO é uma entidade fraca no contexto lógico; ele não possui razão de existir sem um atleta que o realizou e uma prova de origem. Por isso, a relação é forte e identificadora.

3. Cardinalidade e Notação Crow's Foot
||--o{ (Exatamente um para zero ou muitos): Aplicado em PROVA para RESULTADO. Uma prova pode ser cadastrada no sistema antes de acontecer (ter zero resultados), mas cada resultado obrigatoriamente pertence a exatamente uma prova.

||..o{ (Exatamente um para zero ou muitos): Aplicado em SEXO para ATLETA. Todo atleta deve ter um sexo definido no cadastro, mas o sistema pode ter um sexo cadastrado (ex: "Outro") que ainda não possui atletas vinculados.

4. Integridade de Chaves
PK (Primary Key): IDs numéricos auto-incrementais para garantir performance em buscas e joins.

FK (Foreign Key): Garante que o banco de dados não aceite um atleta vinculado a um clube que não existe.

UK (Unique Key): Aplicada em sigla de ESTADO e descricao de SEXO para impedir a criação de registros duplicados com nomes diferentes para a mesma entidade real.
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
        time tempo_final
        int colocacao
        decimal pontos
    }
````
  
