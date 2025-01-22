# Projeto de parsing
Dado que não gosto de fazer nada pequeno, decidi implementar uma logica de formatação de dados, em que através de um sistema de pesos, transformo algo do tipo:
```bash
erro 1234 pacote não instalado pip
erro 1232 pacote não instalado python
erro 12345 pacote não instalado rust
WARN:NoRootBuild 12121
WARN:NoRootBuild 323
INFO:Instalação concluída com sucesso
```

Nisso:
```bash
ERRO 1234, 1232, 12345 pacote não instalado pip, python, rust
WARN 12121, 323 NoRootBuild
INFO Instalação concluída com sucesso
```
No caso, o que o script faz é:
- Separar as mensagens por categoria 
- Realiza o agrupamento de palavras chaves através das categorias, onde:
- - key permanece inalterado se igual.
- - key concatena palavra diferente.

```shell
<<<<<<< HEAD
OUTPUT=$($ORIGINAL_SCRIPT 2>/dev/null) # Recebe tudo que foi cuspido
=======
OUTPUT=$($ORIGINAL_SCRIPT)
>>>>>>> 304accb41acbe9912cca873f23abba1f1058afcb
OUTPUT=$($ORIGINAL_SCRIPT 2>/dev/null) # Recebe a saída do script original

process_out() {
    local output="$1"
    declare -A errors
    declare -A warns
    declare -A infos

    # Processa a saída e agrupa mensagens
    while IFS= read -r linha; do
        if [[ "$linha" =~ [Ee][Rr][Rr][Oo] ]]; then
            id=$(echo "$linha" | grep -oE '[0-9]+')
            mensagem=$(echo "$linha" | sed -E 's/[0-9]+//g' | sed -E 's/[Ee][Rr][Rr][Oo]//I' | xargs)
            [[ -n "$mensagem" ]] && errors["$mensagem"]+="${id}, "
        elif [[ "$linha" =~ [Ww][Aa][Rr][Nn] ]]; then
            id=$(echo "$linha" | grep -oE '[0-9]+')
            mensagem=$(echo "$linha" | sed -E 's/[0-9]+//g' | sed -E 's/[Ww][Aa][Rr][Nn]//I' | xargs)
            [[ -n "$mensagem" ]] && warns["$mensagem"]+="${id}, "
        else
            mensagem=$(echo "$linha" | sed -E 's/[Ii][Nn][Ff][Oo]//I' | xargs)
            [[ -n "$mensagem" ]] && infos["$mensagem"]=1
        fi
    done <<< "$output"

    # Constrói a saída formatada
    local formatted_output=""

    for mensagem in "${!errors[@]}"; do
        ids=$(echo "${errors[$mensagem]}" | sed 's/, $//') # Remove a última vírgula
        formatted_output+="ERRO $ids $mensagem\n"
    done
    for mensagem in "${!warns[@]}"; do
        ids=$(echo "${warns[$mensagem]}" | sed 's/, $//') # Remove a última vírgula
        formatted_output+="WARN $ids $mensagem\n"
    done
    for mensagem in "${!infos[@]}"; do
        formatted_output+="INFO $mensagem\n"
    done

    echo -e "$formatted_output"
}

# Chama a função de processamento e exibe o resultado
RESULTADO=$(process_out "$OUTPUT")
echo -e "$RESULTADO"

```