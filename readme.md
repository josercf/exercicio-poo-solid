# Exercício Prático: SOLID e Clean Code em C#

## 1. Caso de Uso: Sistema de Gerenciamento de Biblioteca

Você foi contratado para desenvolver um sistema simples de gerenciamento de biblioteca que precisa:

- Cadastrar livros com título, autor e ISBN
- Cadastrar usuários com nome e ID
- Realizar empréstimos de livros para usuários
- Calcular multas para atrasos na devolução (R$ 1,00 por dia de atraso)
- Enviar notificações aos usuários sobre empréstimos e multas

## 2. Código Atual (com violações de SOLID e Clean Code)

Aqui está a implementação atual do sistema:

```csharp
using System;
using System.Collections.Generic;

namespace BibliotecaApp
{
    // Classe que faz tudo no sistema
    public class GerenciadorBiblioteca
    {
        private List<Livro> livros = new List<Livro>();
        private List<Usuario> usuarios = new List<Usuario>();
        private List<Emprestimo> emprestimos = new List<Emprestimo>();
        
        // Método que adiciona um livro
        public void AdicionarLivro(string titulo, string autor, string isbn)
        {
            var l = new Livro();
            l.Titulo = titulo;
            l.Autor = autor;
            l.ISBN = isbn;
            l.Disponivel = true;
            livros.Add(l);
            Console.WriteLine("Livro adicionado: " + titulo);
        }
        
        // Método que adiciona um usuário
        public void AdicionarUsuario(string nome, int id)
        {
            var u = new Usuario();
            u.Nome = nome;
            u.ID = id;
            usuarios.Add(u);
            Console.WriteLine("Usuário adicionado: " + nome);
            
            // Enviar email de boas-vindas
            EnviarEmail(u.Nome, "Bem-vindo à Biblioteca", "Você foi cadastrado em nosso sistema!");
        }
        
        // Método que realiza empréstimo
        public bool RealizarEmprestimo(int usuarioId, string isbn, int diasEmprestimo)
        {
            var livro = livros.Find(l => l.ISBN == isbn);
            var usuario = usuarios.Find(u => u.ID == usuarioId);
            
            if (livro != null && usuario != null && livro.Disponivel)
            {
                livro.Disponivel = false;
                var emprestimo = new Emprestimo();
                emprestimo.Livro = livro;
                emprestimo.Usuario = usuario;
                emprestimo.DataEmprestimo = DateTime.Now;
                emprestimo.DataDevolucaoPrevista = DateTime.Now.AddDays(diasEmprestimo);
                emprestimos.Add(emprestimo);
                
                // Enviar email sobre empréstimo
                EnviarEmail(usuario.Nome, "Empréstimo Realizado", 
                    "Você pegou emprestado o livro: " + livro.Titulo);
                
                // Enviar SMS (misturando responsabilidades)
                EnviarSMS(usuario.Nome, "Empréstimo do livro: " + livro.Titulo);
                
                return true;
            }
            
            return false;
        }
        
        // Método que realiza devolução e calcula multa
        public double RealizarDevolucao(string isbn, int usuarioId)
        {
            var emprestimo = emprestimos.Find(e => 
                e.Livro.ISBN == isbn && 
                e.Usuario.ID == usuarioId && 
                e.DataDevolucaoEfetiva == null);
                
            if (emprestimo != null)
            {
                emprestimo.DataDevolucaoEfetiva = DateTime.Now;
                emprestimo.Livro.Disponivel = true;
                
                // Calcular multa (R$ 1,00 por dia de atraso)
                double multa = 0;
                if (DateTime.Now > emprestimo.DataDevolucaoPrevista)
                {
                    TimeSpan atraso = DateTime.Now - emprestimo.DataDevolucaoPrevista;
                    multa = atraso.Days * 1.0;
                    
                    // Enviar email sobre multa
                    EnviarEmail(emprestimo.Usuario.Nome, "Multa por Atraso", 
                        "Você tem uma multa de R$ " + multa);
                }
                
                return multa;
            }
            
            return -1; // Código de erro
        }
        
        // Método para enviar e-mail
        private void EnviarEmail(string destinatario, string assunto, string mensagem)
        {
            // Simulação de envio de e-mail
            Console.WriteLine($"E-mail enviado para {destinatario}. Assunto: {assunto}");
        }
        
        // Método para enviar SMS
        private void EnviarSMS(string destinatario, string mensagem)
        {
            // Simulação de envio de SMS
            Console.WriteLine($"SMS enviado para {destinatario}: {mensagem}");
        }
        
        // Método para buscar todos os livros
        public List<Livro> BuscarTodosLivros()
        {
            return livros;
        }
        
        // Método para buscar todos os usuários
        public List<Usuario> BuscarTodosUsuarios()
        {
            return usuarios;
        }
        
        // Método para buscar todos os empréstimos
        public List<Emprestimo> BuscarTodosEmprestimos()
        {
            return emprestimos;
        }
    }
    
    // Classe de Livro
    public class Livro
    {
        public string Titulo { get; set; }
        public string Autor { get; set; }
        public string ISBN { get; set; }
        public bool Disponivel { get; set; }
    }
    
    // Classe de Usuário
    public class Usuario
    {
        public string Nome { get; set; }
        public int ID { get; set; }
    }
    
    // Classe de Empréstimo
    public class Emprestimo
    {
        public Livro Livro { get; set; }
        public Usuario Usuario { get; set; }
        public DateTime DataEmprestimo { get; set; }
        public DateTime DataDevolucaoPrevista { get; set; }
        public DateTime? DataDevolucaoEfetiva { get; set; }
    }
    
    // Classe Program para testar
    class Program
    {
        static void Main(string[] args)
        {
            var biblioteca = new GerenciadorBiblioteca();
            
            // Adicionar livros
            biblioteca.AdicionarLivro("Clean Code", "Robert C. Martin", "978-0132350884");
            biblioteca.AdicionarLivro("Design Patterns", "Erich Gamma", "978-0201633610");
            
            // Adicionar usuários
            biblioteca.AdicionarUsuario("João Silva", 1);
            biblioteca.AdicionarUsuario("Maria Oliveira", 2);
            
            // Realizar empréstimo
            biblioteca.RealizarEmprestimo(1, "978-0132350884", 7);
            
            // Realizar devolução (com atraso simulado)
            // Note: Em uma aplicação real, você não manipularia as datas desta forma
            double multa = biblioteca.RealizarDevolucao("978-0132350884", 1);
            Console.WriteLine($"Multa por atraso: R$ {multa}");
            
            Console.ReadLine();
        }
    }
}
```

## 3. Atividade para os Alunos

### Parte 1: Identificação de Problemas

Identifique pelo menos 5 violações dos princípios SOLID e boas práticas de Clean Code no código acima. Para cada violação:

1. Indique a classe e o método onde ocorre
2. Explique qual princípio SOLID ou regra de Clean Code está sendo violado
3. Justifique por que isso é considerado uma má prática

### Parte 2: Refatoração

Refatore o código para corrigir as violações identificadas na Parte 1. Seu código refatorado deve:

1. Seguir todos os princípios SOLID:
   - Single Responsibility Principle (SRP)
   - Open/Closed Principle (OCP)
   - Liskov Substitution Principle (LSP)
   - Interface Segregation Principle (ISP)
   - Dependency Inversion Principle (DIP)

2. Aplicar boas práticas de Clean Code:
   - Nomes significativos
   - Funções pequenas e focadas
   - Comentários adequados (apenas quando necessário)
   - Tratamento adequado de erros
   - Organização lógica de código

## 4. Critérios de Avaliação

Sua solução será avaliada com base nos seguintes critérios:

1. Correta identificação das violações SOLID e Clean Code
2. Qualidade da implementação refatorada
3. Aplicação adequada dos princípios SOLID
4. Clareza e organização do código
5. Manutenção da funcionalidade original do sistema

## 5. Entrega

Entregue dois arquivos:
1. Um documento descrevendo as violações encontradas (Parte 1)
2. O código C# refatorado (Parte 2)

Prazo: [DEFINIR DATA]

## 6. Dicas para os Alunos

- Preste atenção especial à classe `GerenciadorBiblioteca` e suas responsabilidades
- Considere o uso de interfaces para implementar o Dependency Inversion Principle
- Pense em como separar as responsabilidades de notificação das operações da biblioteca
- Avalie se as classes atuais estão coesas e têm uma única responsabilidade
