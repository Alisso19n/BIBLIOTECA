# BIBLIOTECA
Trabalho prático,  Disciplina Estrutura de Dados

//  ALISSON E PEDRO


#include <stdio.h>
#include <stdbool.h>
#include <string.h>

#define MAX_LIVROS		100
#define	MAX_RESERVAS	10
#define MAX_USUARIOS	100
#define MAX_EMPRESTIMOS	300

// Estrutura para armazenar data

typedef struct{
    int dia;
    int mes;
    int ano;
} Data;

// Estrutura para armazenar usuario

typedef struct{
    char nome[100];
    char login[10];
    char senha[10];
} Usuario;

// Estrutura para armazenar livro

typedef struct{
    char nome[100];
    char registro[10];
    char autor[100];
    char editora[100];
    bool disponivel;
    Data data_publicacao;
    int qtd_reservas;
    Usuario reservas[MAX_RESERVAS];
} Livro;

// Estrutura para armazenar emprestimo

typedef struct{
	Usuario usuario;
	Livro livro;
	Data data_emprestimo;
} Emprestimo;

// Vetores que armazenam dados (livros, usuarios e emprestimos) do sistema

Livro livros[MAX_LIVROS];
int qtd_livros = 0;

Usuario usuarios[MAX_USUARIOS];
int qtd_usuarios = 0;

Emprestimo emprestimos[MAX_EMPRESTIMOS];
int qtd_emprestimos = 0;

/*
Funcao que dado login e senha retorna o usuario cadastrado com estes dados
Caso nenhum usuario possua os dados, retorna NULL
*/

Usuario * buscarUsuario(char login[10], char senha[10]){
	int i;
	for (i = 0; i < qtd_usuarios; i++){
		if((strcmp(login, usuarios[i].login) == 0) && (strcmp(senha, usuarios[i].senha) == 0)){
			return &usuarios[i];
		}
	}
	return NULL;
}

/*
Funcao que imprime os livros cadastrados no sistema
*/

void listarLivros(){
	int i;
	for (i = 0; i < qtd_livros; i++){
		printf("\n-----------------------------\n");
		printf("Nome da Obra: %s\n", livros[i].nome);
		printf("Registro: %s\n", livros[i].registro);
		printf("Autor: %s\n", livros[i].autor);
		printf("Editora: %s\n", livros[i].editora);
		printf("Data de publicacao: %d/%d/%d\n", livros[i].data_publicacao.dia, livros[i].data_publicacao.mes, livros[i].data_publicacao.ano);
		if((livros[i].disponivel == true) && (livros[i].qtd_reservas == 0)){
			printf("Status: Disponivel\n");
		}else if(livros[i].disponivel == false){
			printf("Status: Emprestado\n");
		}else{
			printf("Status: Lista de Espera\n");
		}
		printf("-----------------------------\n");
	}
}

/*
Funcao que imprime os usuarios cadastrados no sistema
*/

void listarUsuarios(){
	int i;
	for (i = 0; i < qtd_usuarios; i++){
		printf("\n-----------------------------\n");
		printf("Nome: %s\n", usuarios[i].nome);
		printf("Login: %s\n", usuarios[i].login);
		printf("Senha: %s\n", usuarios[i].senha);
		printf("-----------------------------\n");
	}
}

/*
Funcao que imprime os emprestimos ativos do sistema
*/

void listarEmprestimos(){
	int i;
	for (i = 0; i < qtd_emprestimos; i++){
		printf("\n-----------------------------\n");
		printf("Nome do usuario: %s\n", emprestimos[i].usuario.nome);
		printf("Nome da Obra: %s\n", emprestimos[i].livro.nome);
		printf("Registro: %s\n", emprestimos[i].livro.registro);
		printf("Autor: %s\n", emprestimos[i].livro.autor);
		printf("Editora: %s\n", emprestimos[i].livro.editora);
		printf("Data de publicacao: %d/%d/%d\n", emprestimos[i].livro.data_publicacao.dia, emprestimos[i].livro.data_publicacao.mes, emprestimos[i].livro.data_publicacao.ano);
		printf("Data do emprestimo: %d/%d/%d\n", emprestimos[i].data_emprestimo.dia, emprestimos[i].data_emprestimo.mes, emprestimos[i].data_emprestimo.ano);
		printf("-----------------------------\n");
	}
}

/*
Funcao que dado registro retorna o livro cadastrado com este registro
Caso nenhum livro possua este registro, retorna NULL
*/

Livro * buscarLivro(char registro[10]){
	int i;
	for (i = 0; i < qtd_livros; i++){
		if(strcmp(registro, livros[i].registro) == 0){
			return &livros[i];
		}
	}
	return NULL;
}

/*
Funcao que retorna a quantidade de livros emprestados a um determinado usuario
*/

int quantidadeEmprestimo(Usuario usuario){
	int contador = 0, i;
	for (i = 0; i < qtd_emprestimos; i++){
		if((strcmp(usuario.login, emprestimos[i].usuario.login) == 0) && (strcmp(usuario.senha, emprestimos[i].usuario.senha) == 0)){
			contador++;
		}
	}
	return contador;
}

/*
Funcao que busca e atualiza os dados de um determinado livro
*/

void alterarLivro(Livro livro){
	int i;
	for (i = 0; i < qtd_livros; i++){
		if(strcmp(livro.registro, livros[i].registro) == 0){
			livros[i] = livro;
		}
	}
}

/*
Funcao que adiciona usuario na lista de espera de um determinado livro
*/

void adicionarListaEspera(Usuario usuario, Livro livro){
	int opc;
	printf("O usuario deseja entrar na lista de espera?\n0 - Nao\n1 - Sim\n");
	scanf("%d", &opc);
	if(opc == 1){
		livro.reservas[livro.qtd_reservas] = usuario;
		livro.qtd_reservas++;
		alterarLivro(livro);
	}
}

/*
Funcao que realiza um imprestimo a um determinado usuario
*/

void realizarEmprestimo(Usuario usuario){
	char registro[10];
	char aux;
	scanf("%c", &aux);

	printf("Digite o registro do livro: ");
	fgets(registro, 10, stdin);
	registro[strlen(registro) - 1] = '\0';

	Livro * livro = buscarLivro(registro); // busca o livro

	if(livro == NULL){
		printf("Livro nao encontrado!\n");
		return;
	}

	if(quantidadeEmprestimo(usuario) == 3){ // quantidade limite de emprestimos ativos atingida
		printf("Quantidade de emprestimos ja excedida!\n");
		return;
	}

	if(livro->disponivel == false){ // Livro esta em posse de um outro usuario
		printf("Livro indisponivel\n");
		adicionarListaEspera(usuario, *livro);
		return;
	}

	Data data;
	printf("Digite dia: ");
	scanf("%d", &data.dia);
	printf("Digite mes: ");
	scanf("%d", &data.mes);
	printf("Digite ano: ");
	scanf("%d", &data.ano);

	Emprestimo emprestimo;
	emprestimo.usuario = usuario;
	emprestimo.livro = *livro;
	emprestimo.data_emprestimo = data;

	if(livro->qtd_reservas == 0){ // Livro disponivel e lista de espera vazia
		emprestimos[qtd_emprestimos] = emprestimo;
		livro->disponivel = false;
		alterarLivro(*livro);
		qtd_emprestimos++;
		printf("Emprestimo realizado com sucesso");
	}else{ // Livro disponivel mas possui lista de espera
		// Iremos verificar se o usuario esta em primeiro na lista de espera
		if((strcmp(usuario.login, livro->reservas[0].login) == 0) && (strcmp(usuario.senha, livro->reservas[0].senha) == 0)){
			int i;
			for(i = 1; i < livro->qtd_reservas; i++){
				livro->reservas[i - 1] = livro->reservas[i];
			}
			livro->qtd_reservas--;
			livro->disponivel = false;
			alterarLivro(*livro);
			emprestimos[qtd_emprestimos] = emprestimo;
			qtd_emprestimos++;
			printf("Emprestimo realizado com sucesso");
		}else{ // Usuario nao esta em primeiro na lista de espera
			printf("Livro indisponivel\n");
			adicionarListaEspera(usuario, *livro);
		}
	}

}

/*
Funcao que calcula quantidade de dias que o usuario demorou para devolver o livro
Consideramos que todo mes possui 30 dias e todo ano possui 365 dias
*/

int quantidadeDiasEmprestado(Data inicio, Data fim){
	int dias = 0;
	if(inicio.ano == fim.ano){
		if(inicio.mes == fim.mes){
			dias = fim.dia - inicio.dia;
		}else{
			dias = (30 - inicio.dia) + fim.dia + (fim.mes - inicio.mes - 1) * 30;
		}
	}else{
		dias = (30 - inicio.dia) + fim.dia + (12 - inicio.mes - 1) * 30 + fim.mes * 30 + (fim.ano - inicio.ano - 1) * 365;
	}
	return dias;
}

/*
Funcao que realiza uma devolucao de livro
*/

void realizarDevolucao(Usuario usuario){
	char registro[10];

	char aux;
	scanf("%c", &aux);

	printf("Digite o registro do livro: ");
	fgets(registro, 10, stdin);
	registro[strlen(registro) - 1] = '\0';

	Livro * livro = buscarLivro(registro); // Busca o livro

	if(livro == NULL){
		printf("Livro nao encontrado!\n");
		return;
	}

	// Busca o emprestimo a ser devolvido

	int i = 0, j;
	for(i = 0; i < qtd_emprestimos; i++){
		if((strcmp(usuario.login, emprestimos[i].usuario.login) == 0) && (strcmp(usuario.senha, emprestimos[i].usuario.senha) == 0) && (strcmp(livro->registro, emprestimos[i].livro.registro) == 0)){
			break;
		}
	}

	if(i == qtd_emprestimos){
		printf("Emprestimo nao encontrado\n");
		return;
	}

	Data data;
	printf("Digite dia: ");
	scanf("%d", &data.dia);
	printf("Digite mes: ");
	scanf("%d", &data.mes);
	printf("Digite ano: ");
	scanf("%d", &data.ano);

	// Verifica se existe atraso e calcula a multa
	int qtd_dias = quantidadeDiasEmprestado(emprestimos[i].data_emprestimo, data);

	if(qtd_dias > 3){
		printf("Aluno deve pagar a multa de R$ %.2lf \n", 0.5 * (qtd_dias - 3));
	}

	// Remove o emprestimo

	for(j = i + 1; j < qtd_emprestimos; j++){
		emprestimos[j - 1] = emprestimos[j];
	}
	qtd_emprestimos--;

	// Atualiza a disponibilidade do livro

	livro->disponivel = true;
	alterarLivro(*livro);

	printf("Devolucao realizada com sucesso\n");
}

/*
Funcao que ira verificar quais livros estao disponiveis e possuem na primeira posicao da lista de espera o usuario em questao
*/

void visualizarNotificacao(Usuario usuario){
	int i;
	printf("Livros que ficaram disponivel via lista de espera: \n\n");
	for (i = 0; i < qtd_livros; i++){
		if((livros[i].disponivel == true) && (livros[i].qtd_reservas > 0) && ((strcmp(usuario.login, livros[i].reservas[0].login) == 0) && (strcmp(usuario.senha, livros[i].reservas[0].senha) == 0))){
			printf("\n-----------------------------\n");
			printf("Nome da Obra: %s\n", livros[i].nome);
			printf("Registro: %s\n", livros[i].registro);
			printf("Autor: %s\n", livros[i].autor);
			printf("Editora: %s\n", livros[i].editora);
			printf("Data de publicacao: %d/%d/%d\n", livros[i].data_publicacao.dia, livros[i].data_publicacao.mes, livros[i].data_publicacao.ano);
			printf("-----------------------------\n");
		}
	}
}

/*
Funcao do menu usuario
*/

void menuUsuario(){
	char login[10];
	char senha[10];

	char aux;
	scanf("%c", &aux);

	printf("Digite o login: ");
	fgets(login, 10, stdin);
	login[strlen(login) - 1] = '\0';
	printf("Digite a senha: ");
	fgets(senha, 10, stdin);
	senha[strlen(senha) - 1] = '\0';

	Usuario * usuario = buscarUsuario(login, senha);

	if(usuario == NULL){
		printf("Login ou senha invalidos\n");
		return;
	}

	int opc;

	while(1){
		printf("\n\n\n");
		printf("\n####### MENU USUARIO #######\n");
		printf("Digite:\n1- Listar Livros\n2- Realizar Emprestimo\n3- Realizar Devolucao\n4- Visualizar Notificacoes\n5 - Sair\n");
		scanf("%d", &opc);
		if(opc == 1){
			listarLivros();
		}else if(opc == 2){
			realizarEmprestimo(*usuario);
		}else if(opc == 3){
			realizarDevolucao(*usuario);
		}else if(opc == 4){
			visualizarNotificacao(*usuario);
		}else if(opc == 5){
			break;
		}else{
			printf("Op��o Inv�lida!\n");
		}
	}


}

/*
Funcao que cadastra um novo livro
*/

void cadastrarLivro(){
	Livro livro;

	char aux;
	scanf("%c", &aux);

	printf("Nome da Obra: ");
	fgets(livro.nome, 100, stdin);
	livro.nome[strlen(livro.nome) - 1] = '\0';
	printf("Registro: ");
	fgets(livro.registro, 10, stdin);
	livro.registro[strlen(livro.registro) - 1] = '\0';
	printf("Autor: ");
	fgets(livro.autor, 100, stdin);
	livro.autor[strlen(livro.autor) - 1] = '\0';
	printf("Editora: ");
	fgets(livro.editora, 100, stdin);
	livro.editora[strlen(livro.editora) - 1] = '\0';

	Data data;
	printf("Digite dia de publicacao: ");
	scanf("%d", &data.dia);
	printf("Digite mes de publicacao: ");
	scanf("%d", &data.mes);
	printf("Digite ano de publicacao: ");
	scanf("%d", &data.ano);
	livro.data_publicacao = data;
	livro.disponivel = true;
	livro.qtd_reservas = 0;

	// Verifica se o registro ja nao esta vinculado a um livro ja cadastrado no sistema

	Livro * livroAux = buscarLivro(livro.registro);

	if(livroAux != NULL){
		printf("Livro ja cadastrado!\n");
		return;
	}

	// Iremos inserir o livro de forma a manter os livros em ordem alfabetica de seus respctivos nomes

	int i = 0, j;
	for (i = 0; i < qtd_livros; i++){
		if(strcmp(livro.nome, livros[i].nome) < 0){ // Verifica se o nome do livro a ser inserido vem antes do livro cadastrado na posicao i
			break;
		}
	}

	// Desloca parte do vetor em 1 posicao

	for (j = qtd_livros - 1; j >= i; j--){
		livros[j + 1] = livros[j];
	}

	// Insere na posicao correta

	livros[i] = livro;

	qtd_livros++;

	printf("Livro cadastrado com sucesso\n");
}

/*
Funcao que cadastra um novo usuario
*/

void cadastrarUsuario(){
	char aux;
	scanf("%c", &aux);

	Usuario usuario;
	printf("Nome: ");
	fgets(usuario.nome, 100, stdin);
	usuario.nome[strlen(usuario.nome) - 1] = '\0';
	printf("Login: ");
	fgets(usuario.login, 10, stdin);
	usuario.login[strlen(usuario.login) - 1] = '\0';
	printf("Senha: ");
	fgets(usuario.senha, 10, stdin);
	usuario.senha[strlen(usuario.senha) - 1] = '\0';

	// Verifica se este login e senha ja nao estao associados a um outro usuario

	Usuario * usuarioAux = buscarUsuario(usuario.login, usuario.senha);

	if(usuarioAux != NULL){
		printf("Login e senha invalidos\n");
		return;
	}

	usuarios[qtd_usuarios] = usuario;
	qtd_usuarios++;

	printf("Usuario cadastrado com sucesso\n");
}

/*
Funcao do menu administrador
*/

void menuAdministrador(){
	char login[10];
	char senha[10];

	char aux;
	scanf("%c", &aux);

	printf("Digite o login: ");
	fgets(login, 10, stdin);
	login[strlen(login) - 1] = '\0';
	printf("Digite a senha: ");
	fgets(senha, 10, stdin);
	senha[strlen(senha) - 1] = '\0';

	if((strcmp(login, "admin") != 0) || (strcmp(senha, "admin") != 0)){
		printf("Login ou senha invalidos\n");
		return;
	}

	int opc;

	while(1){
		printf("\n\n\n");
		printf("\n####### MENU ADMINISTRADOR #######\n");
		printf("Digite:\n1- Cadastrar Livro\n2- Cadastrar Usuario\n3- Listar Livros\n4- Listar Usuarios\n5- Listar Emprestimos\n6 - Sair\n");
		scanf("%d", &opc);
		if(opc == 1){
			cadastrarLivro();
		}else if(opc == 2){
			cadastrarUsuario();
		}else if(opc == 3){
			listarLivros();
		}else if(opc == 4){
			listarUsuarios();
		}else if(opc == 5){
			listarEmprestimos();
		}else if(opc == 6){
			break;
		}else{
			printf("Op��o Inv�lida!\n");
		}
	}
}

/*
Funcao main que contem o menu principal
*/

int main(){

	int opc;

	while(1){
		printf("\n\n\n");
		printf("\n\n***************************SEJAM BEM VINDOS**************************");
printf("\n\n**************************A BIBLIOTECA*******************************");
printf("\n\n********************************IFBA*********************************\n\n");
		printf("\n####### MENU PRINCIPAL #######\n");
		printf("Digite:\n1- Acesso usuario\n2- Acesso administrador\n3- Sair do sistema\n");
		scanf("%d", &opc);
		if(opc == 1){
			menuUsuario();
		}else if(opc == 2){
			menuAdministrador();
		}else if(opc == 3){
			break;
		}else{
			printf("Op��o Inv�lida!\n");
		}
	}

	return 0;
}
