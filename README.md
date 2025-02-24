/*

Projeto realizado para a disciplina SCC0221 - Introdução à Ciência da Computação I (2023) como parte das atividades avaliativas
propostas durante o semestre.

Este projeto consiste em um mercadinho que realiza a inserção de produtos, vendas, consultas ao estoque,
consultas ao saldo, modificação de preços e aumento de estoque a partir de comandos escolhidos pelo usuário na linha de comando.

Autores:    
            Lucas André - 13673260
            Eduardo Oliveira - 9013182
            João Pedro Viguini T. T. Correa - 14675503
            
            
Orientador: Prof. Dr. Rudinei Goularte.

*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct{

    char nome[30];
    int quantidade;
    float preco;

}Produto;


void add_linha() {
    
    printf("--------------------------------------------------\n");
    
}

// Função para fazer a alocação inicial do vetor da struct Produto

void aloca_estoque (Produto **itens, int capacidade_estoque){
    
    if((*itens = (Produto*) malloc(capacidade_estoque * sizeof(Produto))) == NULL){
        exit(1);
    }
}

void inserir_produto(Produto **itens, int *pos, int *capacidade_estoque, int *quantidade_estoque) {
    
    if(*pos == *capacidade_estoque){
        *capacidade_estoque = *capacidade_estoque + 10;
        if((*itens = (Produto*) realloc(*itens, *capacidade_estoque * sizeof(Produto))) == NULL){
            exit(1);
        }
    }
    
    scanf("%s %d %f", (*itens)[*pos].nome, &(*itens)[*pos].quantidade, &(*itens)[*pos].preco);
    
    (*quantidade_estoque)++;
    
    (*pos)++;
    
}

void aumentar_estoque(Produto *itens){

    int codigo;
    int quantidade;

    scanf("%d %d", & codigo, & quantidade);
    
    if (codigo >= 0) { 
        itens[codigo].quantidade = itens[codigo].quantidade + quantidade;
    }
}

void vender_estoque(Produto *itens, float *saldo){

    int codigo = 0;
    float soma = 0;

    while(codigo != -1){ // Efetua a venda na medida em que os produtos possuem códigos diferentes de -1.

        scanf("%d", &codigo);

        if(codigo >= 0){ // Não considera execução quando o código vale -1.

            if (itens[codigo].quantidade > 0){
                printf("%s %.2f\n",itens[codigo].nome,itens[codigo].preco);
                *saldo = *saldo + itens[codigo].preco;
                soma = soma + itens[codigo].preco;
                itens[codigo].quantidade--; 
            }
            

        }

    }

    printf("Total: %.2f\n",soma);
    add_linha();
}

void modificar_preco(Produto *itens){

    int codigo;
    float preco;

    
    scanf("%d %f", & codigo, & preco);
    
    if (codigo >= 0) { 
        itens[codigo].preco = preco;
    }
}

void consultar_estoque(Produto *itens, int *pos, FILE *fp){

    for(int i = 0; i < *pos;i++){

        printf("%d %s %d\n", i, itens[i].nome , itens[i].quantidade );
        
    }

    add_linha();

}

void consultar_saldo(float *saldo){

    printf("Saldo: %.2f\n", *saldo);
    add_linha();

}

int main(){

    int quantidade_estoque;
    int capacidade_estoque;
    Produto *itens;
    float saldo = 100; // Saldo inicial de 100 reais
    char entrada[3]; 
    int pos = 0; // Posição para a struct ser lida
    FILE *fp;
    
    
    if((fp = fopen("arquivo.dat", "rb")) == NULL){
        
        /*Lê a quantidade de produtos no estoque*/
        scanf("%d", &quantidade_estoque);
        
        /*Lê o saldo*/
        scanf("%f", &saldo);
        
        capacidade_estoque = quantidade_estoque;
        
        aloca_estoque (&itens, capacidade_estoque);

    } else {
        fseek(fp, 0, SEEK_END);
        quantidade_estoque = (ftell(fp) - sizeof(float))/sizeof(Produto);
        rewind(fp);
        
        capacidade_estoque = quantidade_estoque;
        
        aloca_estoque (&itens, capacidade_estoque);
        
        fread(&saldo, sizeof(saldo), 1, fp);
        fread(itens, sizeof(Produto), quantidade_estoque, fp);
        pos = quantidade_estoque;
        fclose(fp);
    }
    
    
    
    do {
        
        scanf("%s", entrada); // Lê o comando do usuário

        /*Inserir Produto (IP)*/
        
        if(strcmp(entrada,"IP") == 0) {

            inserir_produto(&itens, &pos, &capacidade_estoque, &quantidade_estoque);
            
        }

        /*Aumentar Estoque (AE)*/

        if(strcmp(entrada,"AE") == 0){

            aumentar_estoque(itens);

        }

        /*Modificar Preço (MP)*/

        if(strcmp(entrada,"MP") == 0){

            modificar_preco(itens);

        }

        /*Vender Estoque (VE)*/

        if(strcmp(entrada,"VE") == 0){

            vender_estoque(itens,&saldo);
        }

        /*Consultar Estoque (CE)*/

        if(strcmp(entrada,"CE") == 0){

            consultar_estoque(itens,&pos,fp);
            
        }

        /*Consultar Saldo (CS)*/

        if(strcmp(entrada,"CS") == 0){

            consultar_saldo(&saldo);

        }
    
    } while (strcmp(entrada,"FE") != 0); // Finalizar Execução (FE)
    
    fp = fopen("arquivo.dat", "wb");
    fwrite(&saldo, sizeof(saldo), 1, fp);
    fwrite(itens, sizeof(Produto), quantidade_estoque, fp);
    fclose(fp);
    
    free(itens);
    
    return 0;
}
