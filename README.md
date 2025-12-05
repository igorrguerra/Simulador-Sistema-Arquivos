Simulador de Sistema de Arquivos

Resumo

Este trabalho propõe o desenvolvimento de um simulador para compreender um sistema de arquivos. O projeto foca na implementação prática de estruturas de diretórios e arquivos, bem como na aplicação de técnicas de Journaling para integridade de dados.

Introdução

O gerenciamento eficiente de arquivos é crucial para o funcionamento dos sistemas operacionais. Para isso, entender como é montado e organizado um sistema é a base para a compreensão dos sistemas operacionais. Este projeto simula um ambiente controlado onde operações de disco são emuladas via software.

Objetivo

Desenvolver um simulador de sistema de arquivos em Java que implemente funcionalidades básicas de manipulação de arquivos e diretórios, com suporte a Journaling para garantir a integridade dos dados. Este simulador deverá permitir a criação de um arquivo que simule o sistema de arquivos e realizar operações como copiar, apagar, renomear arquivos e diretórios, bem como listar o conteúdo de um diretório.

Metodologia

O simulador foi desenvolvido utilizando a linguagem de programação Java. O sistema opera em um "Modo Shell" (Linha de Comando), onde o usuário interage através de comandos textuais.

A persistência dos dados é simulada através da serialização de objetos Java. O estado completo do sistema de arquivos (árvore de diretórios e arquivos) é salvo em um arquivo físico no computador hospedeiro chamado filesystem.dat.

As operações implementadas seguem o padrão de chamadas de métodos encapsuladas em blocos de tratamento de erro e registro de log (Journaling).

Parte 1: Introdução ao Sistema de Arquivos com Journaling

Descrição do Sistema de Arquivos

Um sistema de arquivos é o componente do sistema operacional responsável por controlar como os dados são armazenados e recuperados. Sem um sistema de arquivos, as informações colocadas em um meio de armazenamento seriam um grande corpo de dados sem maneira de saber onde um pedaço de informação termina e o próximo começa. Ele gerencia o espaço em disco, mantém a hierarquia de diretórios e armazena metadados (como nomes de arquivos, datas de criação e permissões).

Journaling

O Journaling é uma técnica utilizada para garantir a integridade dos dados em caso de falhas no sistema (como quedas de energia). Antes de realizar alterações reais nos metadados ou dados do sistema de arquivos, o sistema registra a intenção dessa mudança em uma área dedicada chamada "Journal" (ou Log).

Tipos de Journaling:

Write-Ahead Logging: Registra os metadados e/ou dados no journal antes de escrever na estrutura principal. É o modelo simplificado adotado neste simulador.

Ordered Mode: Apenas os metadados são logados, mas os dados são escritos antes dos metadados serem marcados como "commit" no journal.

Log-Structured: O sistema de arquivos inteiro é estruturado como um log circular.

Neste projeto, implementamos um Write-Ahead Log simplificado, onde cada operação (Criar, Deletar, Renomear) é registrada com um timestamp e o status da operação (START, COMMIT, ROLLBACK).

Parte 2: Arquitetura do Simulador

Estrutura de Dados

Para representar o sistema de arquivos em memória, utilizamos uma estrutura de árvore:

Classe Abstrata FSNode: A base para qualquer entidade no sistema. Contém propriedades comuns como name, createdAt e referência ao parent (diretório pai).

Classe Directory (Estende FSNode): Representa uma pasta.

Utiliza um HashMap<String, FSNode> para armazenar seus filhos (arquivos ou subdiretórios), permitindo busca rápida por nome.

Classe MyFile (Estende FSNode): Representa um arquivo.

Contém um campo para conteúdo (texto).

Journaling

O Journaling é implementado na classe Journal.

Estrutura do Log: Uma List<String> que armazena o histórico de operações.

Fluxo:

O comando é recebido pelo Shell.

O Journal registra START <Operação>.

A operação é tentada no FileSystem.

Se bem-sucedida, o Journal registra COMMIT.

Se falhar, o Journal registra ROLLBACK e a exceção.

O estado atual é salvo em disco (filesystem.dat) após cada operação crítica.

Parte 3: Implementação em Java

A implementação foi consolidada na classe principal FileSystemSimulator para facilitar a execução, contendo classes internas estáticas para a lógica do domínio.

Principais Componentes:

Classe FileSystemSimulator (Main):

Gerencia o loop principal (while(true)).

Faz o parsing dos comandos digitados (mkdir, ls, etc.).

Chama o método executeWithJournal que envolve a execução real.

Classe FileSystem:

Mantém a referência para o diretório root e currentDirectory.

Implementa a lógica de navegação (cd).

Implementa operações CRUD:

mkdir(String name)

touch(String name)

rm(String name) e rmdir(String name)

mv(String old, String new)

cp(String src, String dest)

Persistência:

Métodos loadFileSystem() e saveFileSystem() utilizam ObjectInputStream e ObjectOutputStream para gravar o estado da memória no arquivo filesystem.dat.

Parte 4: Instalação e Funcionamento

Passo a Passo para Execução

Pré-requisitos: Certifique-se de ter o Java JDK instalado (versão 8 ou superior).

Download: Baixe o arquivo FileSystemSimulator.java.

Compilação:
Abra o terminal ou prompt de comando na pasta do arquivo e digite:

javac FileSystemSimulator.java


Execução:
Ainda no terminal, execute:

java FileSystemSimulator


Uso:
O sistema abrirá um prompt simulado /$. Você pode digitar os comandos:

mkdir pasta1 (Cria diretório)

touch arquivo.txt (Cria arquivo)

ls (Lista conteúdo)

cd pasta1 (Entra na pasta)

cd .. (Volta uma pasta)

cp arquivo.txt copia.txt (Copia arquivo)

mv copia.txt novo_nome.txt (Renomeia)

rm arquivo.txt (Remove arquivo)

status (Verifica o Log do Journaling)

exit (Salva e sai)

Resultados Esperados

O simulador demonstra com sucesso a hierarquia de arquivos e a importância do Journaling. Ao reiniciar o programa, as alterações anteriores persistem (graças ao arquivo filesystem.dat), simulando um disco rígido real. O log de journaling permite auditar todas as ações realizadas, essencial para depuração e integridade em sistemas reais.
