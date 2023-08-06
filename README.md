# marcus-lang
Uma lingaguem inspirada nas linguagens NATURAL e COBOL que compila para JavaScript.
No futuro será possível criar microserviços em Marcus, que vão rodar em nodejs.


# Exemplo 

busca-pacientes.mar

```cobol

PROGRAM BUSCA-PACIENTES
            WITH-INPUT       T-ENTRADA
            WITH-OUTPUT      T-SAIDA

    TYPE T-ENTRADA
        VAR UF    STRING
    END-TYPE

    TYPE T-PACIENTE
        VAR ID    NUMBER
        VAR NOME  STRING
    END-TYPE

    TYPE T-SAIDA
        CODIGO    NUMBER
        MSG       STRING
        DADOS     LIST OF T-PACIENTE
    END-TYPE
    
    VAR DB012             STRING = "DB012"

    VAR SQL               STRING
    VAR UF                STRING
    VAR PACIENTE          T-PACIENTE
    VAR PACIENTES         LIST OF T-PACIENTE
    VAR I                 INT = 0

    VAR CODIGO-ERRO       NUMBER
    VAR MSG-ERRO          STRING

    VAR SAIDA             T-SAIDA

    PROCEDURE MAIN

        PERFORM INICIALIZAR-VARIAVEIS
        PERFORM CONECTAR-BANCO-DE-DADOS
        PERFORM BUSCAR-PACIENTES
        PERFORM LIBERAR-RECURSOS
        PERFORM RETORNAR-COM-SUCESSO

    END-PROCEDURE

    PROCEDURE INICIALIZAR-VARIAVEIS
        
        MOVE "
            SELECT * FROM DB012.PACIENTES
            WHERE UF = ?
        "  TO SQL

        MOVE "CE"   TO UF
        
    END-PROCEDURE

    PROCEDURE CONECTAR-BANCO-DE-DADOS

        DB-CONNECT DB012

        IF DB-CONNECT-ERROR THEN

            MOVE 1  TO SAIDA.CODIGO
            MOVE "Erro ao conectar com a base de dados"
                    TO SAIDA.MSG

            PERFORM RETORNAR-COM-ERRO

        END-IF 

    END-PROCEDURE

    PROCEDURE BUSCAR-PACIENTES
        
        DB-EXEC SQL ON DB012
        WITH UF

        IF DB-EXEC-ERROR THEN
            
            MOVE 2  TO SAIDA.CODIGO
            MOVE "Erro ao buscar lista de pacientes"
                    TO SAIDA.MSG

            PERFORM RETORNAR-COM-ERRO

        END-IF

        FOR I = 1 TO DB-NUM-ROWS STEP 1

            RESET PACIENTE

            MOVE DB-ROW.ID    TO PACIENTE.CODIGO
            MOVE DB-ROW.NOME  TO PACIENTE.NOME

            PUSH PACIENTE     TO PACIENTES

        END-FOR

    END-PROCEDURE

    PROCEDURE RETORNAR-COM-SUCESSO

        MOVE 0         TO SAIDA.CODIGO
        MOVE "Operação realizada com sucesso" TO SAIDA.MSG
        MOVE PACIENTES TO SAIDA.DADOS

        RETURN SAIDA

    END-PROCEDURE

    PROCEDURE RETORNAR-COM-ERRO

        MOVE CODIGO-ERRO         TO SAIDA.CODIGO
        MOVE MSG-ERRO            TO SAIDA.MSG
        MOVE []                  TO SAIDA.DADOS

        PERFORM LIBERAR-RECURSOS

        RETURN SAIDA

    END-PROCEDURE

    PROCEDURE LIBERAR-RECURSOS
        DB-CLOSE DB012
    END-PROCEDURE
    

END-PROGRAM


```


