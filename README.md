# Pim
Desenvolvimento de um sistema composto por múltiplos programas em modo console para  otimizar as operações de um hortifruti.


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <conio.h>

#define MAX_LOGIN 50
#define MAX_SENHA 6
#define MAX_CODIGO 10
#define MAX_PRODUTO 20
#define MAX_FORNECEDOR 20
#define MAX_VALIDADE 10
#define MAX_QUANTIDADE 5
#define MAX_PRECO 10

#define CLEAN_BUFF do{ int c; while((c = getchar()) != '\n' && c != EOF);}while(0)

char usuarioLogado[50] = "Desconhecido";

void pausa(int segundos) {
    int tempoinicial = time(NULL);
    while (time(NULL) - tempoinicial < segundos);
}

int usuario(FILE* file, char* user, char* senha) {
    char tmplogin[MAX_LOGIN];
    char tmpsenha[MAX_SENHA];

    fscanf(file, "%s", tmplogin);

    while (!feof(file)) {
        if (!strcmp(tmplogin, user)) {
            fscanf(file, "%s", tmpsenha);

            if (!strcmp(tmpsenha, senha))
                return 1;
        } else {
            fscanf(file, "%*s");
        }
        fscanf(file, "%s", tmplogin);
    }
    return 0;
}

char* CriaSenha() {
     register int i;
    char* senha = (char*)malloc(sizeof *senha * MAX_SENHA);

    for(i = 0; i < MAX_SENHA; i++) {
        senha[i] = getch();
        if(senha[i] == '\r')
            break;
        else
            printf("*");
    }
    senha[i] = '\0';

    return senha;

}
void realizarLogin() {
    printf("Digite o nome do usu rio para login: ");
    fgets(usuarioLogado, sizeof(usuarioLogado), stdin);
    strtok(usuarioLogado, "\n"); // Remove o '\n' do final da string
}

int obterNumeroPedido() {
    FILE* contadorFile = fopen("contador_pedido.txt", "r");
    int numeroPedido = 1;

    if (contadorFile != NULL) {
        fscanf(contadorFile, "%d", &numeroPedido);
        fclose(contadorFile);
    }

    return numeroPedido;
}

void atualizarNumeroPedido(int numeroPedido) {
    FILE* contadorFile = fopen("contador_pedido.txt", "w");

    if (contadorFile == NULL) {
        perror("Erro ao abrir o arquivo contador_pedido.txt");
        return;
    }

    fprintf(contadorFile, "%d", numeroPedido);
    fclose(contadorFile);
}

void exibirTodosProdutos() {
    FILE* produtosfile = fopen("produtos.txt", "r");
    if (produtosfile == NULL) {
        perror("Erro ao abrir o arquivo");
        return;
    }

    char linha[100];
    printf("Lista de Produtos:\n");
    while (fgets(linha, sizeof(linha), produtosfile)) {
        printf("%s", linha);
    }

    fclose(produtosfile);
}

void buscarPorCodigo(const char* codigoBusca) {
    FILE* produtosfile = fopen("produtos.txt", "r");
    if (produtosfile == NULL) {
        perror("Erro ao abrir o arquivo");
        return;
    }

    char codigo[MAX_CODIGO], produto[MAX_PRODUTO], fornecedor[MAX_FORNECEDOR];
    char validade[MAX_VALIDADE], quantidade[MAX_QUANTIDADE], preco[MAX_PRECO];
    int encontrado = 0;

    while (fscanf(produtosfile, "codigo:%s\nproduto:%s\nfornecedor:%s\nvalidade:%s\nquantidade:%s\npreco:%s\n\n",
                  codigo, produto, fornecedor, validade, quantidade, preco) == 6) {
        if (strcmp(codigo, codigoBusca) == 0) {
            printf("\nProduto encontrado:\n");
            printf("Codigo: %s\n", codigo);
            printf("Produto: %s\n", produto);
            printf("Fornecedor: %s\n", fornecedor);
            printf("Validade: %s\n", validade);
            printf("Quantidade: %s\n", quantidade);
            printf("Preco: %s\n", preco);
            encontrado = 1;
            break;
        }
    }

    if (!encontrado) {
        printf("Produto com o codigo %s nao encontrado.\n", codigoBusca);
    }

    fclose(produtosfile);
}

void adicionarProduto() {
    FILE* produtosfile;
    char codigo[MAX_CODIGO], produto[MAX_PRODUTO], fornecedor[MAX_FORNECEDOR];
    char validade[MAX_VALIDADE], quantidade[MAX_QUANTIDADE], preco[MAX_PRECO];

    printf("codigo: ");
    fgets(codigo, MAX_CODIGO, stdin);
    strtok(codigo, "\n");

    printf("produto: ");
    fgets(produto, MAX_PRODUTO, stdin);
    strtok(produto, "\n");

    printf("fornecedor: ");
    fgets(fornecedor, MAX_FORNECEDOR, stdin);
    strtok(fornecedor, "\n");

    printf("validade: ");
    fgets(validade, MAX_VALIDADE, stdin);
    strtok(validade, "\n");

    printf("quantidade: ");
    fgets(quantidade, MAX_QUANTIDADE, stdin);
    strtok(quantidade, "\n");

    printf("preco: ");
    fgets(preco, MAX_PRECO, stdin);
    strtok(preco, "\n");

    produtosfile = fopen("produtos.txt", "a+");
    if (produtosfile == NULL) {
        perror("Erro ao abrir o arquivo");
        return;
    }

    fprintf(produtosfile, "codigo:%s\nproduto:%s\n", codigo, produto);
    fprintf(produtosfile, "fornecedor:%s\nvalidade:%s\n", fornecedor, validade);
    fprintf(produtosfile, "quantidade:%s\npreco:%s\n\n", quantidade, preco);
    fclose(produtosfile);

    printf("Produto adicionado com sucesso!\n");
    pausa(2);
}
void atualizarEstoque(const char* codigoBusca) {
    FILE* produtosfile = fopen("produtos.txt", "r");
    FILE* tempFile = fopen("temp.txt", "w");
    if (produtosfile == NULL || tempFile == NULL) {
        perror("Erro ao abrir o arquivo");
        return;
    }

    char codigo[MAX_CODIGO], produto[MAX_PRODUTO], fornecedor[MAX_FORNECEDOR];
    char validade[MAX_VALIDADE], quantidade[MAX_QUANTIDADE], preco[MAX_PRECO];
    int encontrado = 0;

    while (fscanf(produtosfile, "codigo:%s\nproduto:%s\nfornecedor:%s\nvalidade:%s\nquantidade:%s\npreco:%s\n\n",
                  codigo, produto, fornecedor, validade, quantidade, preco) == 6) {
        if (strcmp(codigo, codigoBusca) == 0) {
            // Produto encontrado, atualize a quantidade
            encontrado = 1;

            printf("Atualize os dados do produto:\n");
            printf("produto: ");
            fgets(produto, MAX_PRODUTO, stdin);
            strtok(produto, "\n");

            printf("fornecedor: ");
            fgets(fornecedor, MAX_FORNECEDOR, stdin);
            strtok(fornecedor, "\n");

            printf("validade: ");
            fgets(validade, MAX_VALIDADE, stdin);
            strtok(validade, "\n");

            printf("quantidade: ");
            fgets(quantidade, MAX_QUANTIDADE, stdin);
            strtok(quantidade, "\n");

            printf("preco: ");
            fgets(preco, MAX_PRECO, stdin);
            strtok(preco, "\n");
        }

        // Salva o produto atual (modificado ou n o) no arquivo tempor rio
        fprintf(tempFile, "codigo:%s\nproduto:%s\n", codigo, produto);
        fprintf(tempFile, "fornecedor:%s\nvalidade:%s\n", fornecedor, validade);
        fprintf(tempFile, "quantidade:%s\npreco:%s\n\n", quantidade, preco);
    }

    fclose(produtosfile);
    fclose(tempFile);

    if (encontrado) {
        // Substitui o arquivo original pelo tempor rio
        remove("produtos.txt");
        rename("temp.txt", "produtos.txt");
        printf("Estoque atualizado com sucesso!\n");
        pausa(3);
    } else {
        // Remove o arquivo tempor rio se o produto n o foi encontrado
        remove("temp.txt");
        printf("Produto com o codigo %s nao encontrado.\n", codigoBusca);
        pausa(3);
    }
}
void registrarVenda() {
    char respostaPedido, respostaItem;
    int numeroPedido = obterNumeroPedido();

    do {
        FILE* vendasFile = fopen("vendas.txt", "a");
        if (vendasFile == NULL) {
            perror("Erro ao abrir o arquivo de vendas");
            return;
        }
        printf("\nUsuario logado: %s\n", usuarioLogado);
        printf("Iniciando pedido #%d\n", numeroPedido);
        float totalPedido = 0.0;
        int item = 1;

        fprintf(vendasFile, "______________________________________________\n______________________________________________\n");
        fprintf(vendasFile, "Pedido# %d\n", numeroPedido);
        fprintf(vendasFile, "Usuario: %s\n", usuarioLogado);
        fprintf(vendasFile, "______________________________________________\n______________________________________________\n");
        fprintf(vendasFile, "Item\tCodigo\tProduto\tQtde\tVl Unit\tTotal\n");

        // Array para armazenar os itens do pedido
        char itensRegistrados[100][200];
        int itemCount = 0;

        // In cio do loop para registrar itens do pedido
        do {
            FILE* produtosfile = fopen("produtos.txt", "r");
            FILE* tempFile = fopen("temp.txt", "w");

            char codigoBusca[MAX_CODIGO], codigo[MAX_CODIGO], produto[MAX_PRODUTO], fornecedor[MAX_FORNECEDOR];
            char validade[MAX_VALIDADE], quantidade[MAX_QUANTIDADE], preco[MAX_PRECO];
            int quantidadeEstoque, quantidadeVendida;
            float precoUnitario, totalVenda;
            int encontrado = 0;

            printf("\nDigite o codigo do produto para venda: ");
            fgets(codigoBusca, MAX_CODIGO, stdin);
            strtok(codigoBusca, "\n");

            printf("Digite a quantidade vendida: ");
            scanf("%d", &quantidadeVendida);
            getchar();

            while (fscanf(produtosfile, "codigo:%s\nproduto:%s\nfornecedor:%s\nvalidade:%s\nquantidade:%s\npreco:%s\n\n",
                          codigo, produto, fornecedor, validade, quantidade, preco) == 6) {
                if (strcmp(codigo, codigoBusca) == 0) {
                    encontrado = 1;
                    quantidadeEstoque = atoi(quantidade);
                    precoUnitario = atof(preco);

                    if (quantidadeVendida > quantidadeEstoque) {
                        printf("Quantidade em estoque insuficiente.\n");
                        fclose(produtosfile);
                        fclose(tempFile);
                        fclose(vendasFile);
                        remove("temp.txt");
                        pausa(3);
                        return;
                    }

                    totalVenda = quantidadeVendida * precoUnitario;
                    totalPedido += totalVenda;
                    quantidadeEstoque -= quantidadeVendida;
                    sprintf(quantidade, "%d", quantidadeEstoque);


                    fprintf(vendasFile, "%d\t%s\t%s\t%d\t%.2f\t%.2f\n", item, codigo, produto, quantidadeVendida, precoUnitario, totalVenda);

                    // Salva o item no array de itens registrados
                    sprintf(itensRegistrados[itemCount++],
                            "Item: %d | Codigo: %s | Produto: %s | Qtde: %d | Preco Unit: %.2f | Total: %.2f",
                            item++, codigo, produto, quantidadeVendida, precoUnitario, totalVenda);
                }

                fprintf(tempFile, "codigo:%s\nproduto:%s\n", codigo, produto);
                fprintf(tempFile, "fornecedor:%s\nvalidade:%s\n", fornecedor, validade);
                fprintf(tempFile, "quantidade:%s\npreco:%s\n\n", quantidade, preco);
            }

            fclose(produtosfile);
            fclose(tempFile);

            if (encontrado) {
                remove("produtos.txt");
                rename("temp.txt", "produtos.txt");
            } else {
                remove("temp.txt");
                printf("Produto com o codigo %s nao encontrado.\n", codigoBusca);
                pausa(3);
            }

            // Exibe todos os itens registrados no pedido
            system("cls");
            printf("\nItens registrados neste pedido ate agora:\n");
            for (int i = 0; i < itemCount; i++) {
                printf("%s\n", itensRegistrados[i]);
                printf("\nTotal do pedido: %.2f\n", totalPedido);
            }

            printf("\nDeseja adicionar outro item ao pedido? (s/n): ");
            scanf(" %c", &respostaItem);
            getchar();

        } while (respostaItem == 's' || respostaItem == 'S');

        // Verifica se houve itens registrados
        if (itemCount > 0) {
            // Exibe os itens registrados antes de calcular o total
            system("cls");
            printf("\nResumo do pedido #%d:\n", numeroPedido);
            for (int i = 0; i < itemCount; i++) {
                printf("%s\n", itensRegistrados[i]);
            }

            char metodoPagamento[20];
            printf("\nTotal do pedido: %.2f\n", totalPedido);
            printf("\nDigite o metodo de pagamento (dinheiro, cartao, pix, etc.): ");
            fgets(metodoPagamento, 20, stdin);
            metodoPagamento[strcspn(metodoPagamento, "\n")] = 0;  // Remove a quebra de linha

            // Registra o m todo de pagamento junto com o total do pedido
            fprintf(vendasFile, "\nTotal do pedido: %.2f\n", totalPedido);
            fprintf(vendasFile, "Metodo de pagamento: %s\n\n", metodoPagamento);
            system("cls");
            printf("\nTotal do pedido: %.2f\n", totalPedido);
            printf("Metodo de pagamento: %s\n", metodoPagamento);

            fclose(vendasFile);

            numeroPedido++;
            atualizarNumeroPedido(numeroPedido);  // Incrementa o contador de pedidos
        } else {
            system("cls");
            printf("Nenhum item registrado neste pedido. Pedido nao sera registrado.\n");
            fclose(vendasFile);
            // N o incrementa o n mero do pedido
        }

        printf("Deseja registrar um novo pedido? (s/n): ");
        scanf(" %c", &respostaPedido);
        getchar();

    } while (respostaPedido == 's' || respostaPedido == 'S');
}

void RegistrosDasVendas() {
    FILE* vendasFile = fopen("vendas.txt", "r");
    if (vendasFile == NULL) {
        perror("Erro ao abrir o arquivo de vendas");
        return;
    }

    char linha[200];
    while (fgets(linha, sizeof(linha), vendasFile)) {
        printf("%s", linha);
    }

    fclose(vendasFile);
}
typedef struct{
    char ChaveDeAcesso[10];
}acesso;acesso A[1];

int main() {
    FILE* fpIN;
    int opcao,opcaoL,opcaoF,opcaoG,opcaoB;
    int sair;
    char *user, *senha, *confirmaSenha, *ChaveDeAcesso;
    user = (char*)malloc(sizeof *user * MAX_LOGIN);
    senha = (char*)malloc(sizeof *senha * MAX_SENHA);
    confirmaSenha = (char*)malloc(sizeof *confirmaSenha * MAX_SENHA);
    char codigoBusca[MAX_CODIGO];

    strcpy(A[0].ChaveDeAcesso,"1234");

        do {
        sair = 0;
        system("cls");
        printf("______________________________________________\n______________________________________________");
        printf("\n\tEscolha uma opcao:\n");
        printf("______________________________________________\n______________________________________________");
        printf("\n1. Login\n2. Cadastrar Usuario\n3. Sair\n");

        scanf("%d", &opcao);
        CLEAN_BUFF;

        switch (opcao) {
            case 1:
                system("cls");
                printf("1. Funcionario\n2. Gerente\n3. voltar\n");
                scanf("%d", &opcaoL);
                CLEAN_BUFF;

                switch(opcaoL){
                case 1:
                    system("cls");
                    printf("usuario: ");
                    fgets(user, MAX_LOGIN, stdin);
                    strtok(user,"\n");
                    printf("senha: ");
                    senha = CriaSenha();

                    fpIN = fopen("usuarios.txt", "a+");
                    if (usuario(fpIN, user, senha)) {
                        strncpy(usuarioLogado, user, sizeof(usuarioLogado) - 1);
                        printf("\nBem vindo %s.\n", usuarioLogado);
                        pausa(4);

                         while(!sair) {
                            system("cls");
                            printf("______________________________________________\n______________________________________________");
                            printf("\n\tEscolha uma opcao\n");
                            printf("______________________________________________\n______________________________________________");
                            printf("\n1. Registrar venda\n2. Registro das vendas\n3. Voltar\n");
                            scanf("%d", &opcaoF);
                            CLEAN_BUFF;

                            switch(opcaoF) {

                        case 1:
                            system("cls");
                            registrarVenda();
                            break;
                        case 2:
                system("cls");
                RegistrosDasVendas();
                printf("Digite enter para voltar.");
                getchar();
                break;
                        case 3:
                            sair = 1;
                            break;
                        default:
                            system("cls");
                            printf("Opcao invalida. Tente novamente.\n");
                            pausa(3);
                            break;
                            }
                            }
                    }
                    else
                        printf("\nusuario nao registrado\n");

                    fclose(fpIN);
                    free(senha);

                    break;
                case 2:
                    system("cls");
                    printf("Chave de acesso:");
                    ChaveDeAcesso = CriaSenha();

                    if (strcmp(ChaveDeAcesso, "1234") == 0) {
                            while(!sair) {
                            system("cls");
                            printf("______________________________________________\n______________________________________________");
                            printf("\n\tEscolha uma opcao:\n");
                            printf("______________________________________________\n______________________________________________");
                            printf("\n1. Adicionar Produto\n2. Buscar Produto\n3. Atualizar Estoque\n4. Registro de Vendas\n5. Voltar\n");
                            scanf("%d", &opcaoG);
                            CLEAN_BUFF;

                            switch(opcaoG) {

                            case 1:
                                system("cls");
                                adicionarProduto();
                                break;
                            case 2:
                system("cls");
                printf("1. Mostrar Todos Produtos\n2. Buscar pelo Codigo\n");
                scanf("%d", &opcaoB);
                CLEAN_BUFF;

                if (opcaoB == 1) {
                    system("cls");
                    exibirTodosProdutos();
                    printf("Aperte enter para voltar.");
                    getchar();
                } else if (opcaoB == 2) {
                    system("cls");
                    printf("Digite o codigo do produto para buscar: ");
                    fgets(codigoBusca, MAX_CODIGO, stdin);
                    strtok(codigoBusca, "\n");
                    buscarPorCodigo(codigoBusca);
                    printf("\nAperte enter para voltar.");
                    getchar();
                } else {
                    printf("Opcao invalida. Tente novamente.\n");
                    pausa(3);
                }
                break;
                            case 3:
                system("cls");
                printf("Digite o codigo do produto para atualizar: ");
                fgets(codigoBusca, MAX_CODIGO, stdin);
                strtok(codigoBusca, "\n");
                atualizarEstoque(codigoBusca);
                break;
                            case 4:
                system("cls");
                RegistrosDasVendas();
                printf("Digite enter para voltar.");
                getchar();
                break;
                            case 5:
                                sair = 1;
                                break;
                            default:
                                system("cls");
                                printf("Opcao invalida. Tente novamente.\n");
                                pausa(3);
                                break;
                            }
                            }
                    }else {
                        system("cls");
                        printf("Acesso negado.Aperte enter para voltar.");
                        getchar();
                    }
                    break;
                case 3:
                    break;
                default:
                    system("cls");
                printf("Opcao invalida. Tente novamente.\n");
                pausa(3);
                break;
                }
                break;
            case 2:
    system("cls");
    printf("Chave de acesso:");
    ChaveDeAcesso = CriaSenha();

    if (strcmp(ChaveDeAcesso, "1234") == 0) {
        system("cls");
        printf("Usuario:");
        fgets(user, MAX_LOGIN, stdin);
        strtok(user, "\n");

        do {
            printf("Senha:");
            senha = CriaSenha();
            printf("\nConfirmacao de senha:");
            confirmaSenha = CriaSenha();
            printf("\n");

            if (!strcmp(senha, confirmaSenha))
                break;
            else
                printf("As senhas nao sao iguais. Tente novamente.\n");
        } while (1);

        // Abrir o arquivo no modo "a+" para leitura e escrita
        fpIN = fopen("usuarios.txt", "a+");
        if (fpIN == NULL) {
            perror("Erro ao abrir o arquivo de usuarios");
        } else {
            // Verificar se o usu rio j  existe
            rewind(fpIN);  // Retorna ao in cio do arquivo para leitura
            if (usuario(fpIN, user, senha)) {
                printf("\nUsuario ja registrado.\n");
                pausa(3);
            } else {
                // Escrever o novo usu rio no arquivo
                fprintf(fpIN, "%s %s\n", user, senha);
                printf("\nUsuario registrado com sucesso.\n");
                pausa(3);
            }
            fclose(fpIN);
        }

        free(senha);
        free(confirmaSenha);
    } else {
        printf("\nAcesso negado. Aperte enter para voltar.");
        getchar();
    }
    break;
            case 3:
                system("cls");
                printf("Encerrando o programa\n");
                pausa(3);
                free(user);
                return 0;

            default:
                system("cls");
                printf("Opcao invalida. Tente novamente.\n");
                pausa(3);
                break;
        }
    }while(1);
}

