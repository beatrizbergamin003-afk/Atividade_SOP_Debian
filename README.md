## Introdução

# Atividade Configuração Avançada do Servidor

## Introdução

Este relatório apresenta o processo de configuração avançada de um servidor Linux utilizando o Samba para compartilhamento de arquivos em rede. Durante a atividade, foram realizadas configurações de usuários, permissões de acesso, compartilhamento de pastas e testes de conexão entre a máquina servidor e a máquina cliente.

O objetivo principal foi compreender como funciona o gerenciamento de acessos em um ambiente de rede Linux, garantindo diferentes níveis de permissão para cada usuário criado.

---

# Etapa 1 – Configuração Inicial da Rede

Nesta etapa foi realizado o processo de identificação do endereço IP da máquina servidor e o teste de conectividade entre as máquinas utilizando o comando `ping`.

Esse procedimento foi importante para confirmar que a máquina cliente conseguia se comunicar corretamente com o servidor através da rede.
<img width="1283" height="884" alt="image" src="https://github.com/user-attachments/assets/e56a3a2f-5251-4361-b173-c50e04eff6af" />

Teste de conectividade entre cliente e servidor utilizando o comando ping
<img width="1285" height="890" alt="image" src="https://github.com/user-attachments/assets/22827f5b-bf2d-4824-96ea-ea4dee66d871" />

# Etapa 2 – Criação dos Usuários no Linux

Nesta fase foram criados os usuários `admin_ti`, `suporte_ti` e `visitante_ti` utilizando comandos no terminal Linux. Cada usuário recebeu um tipo específico de permissão:

- `admin_ti`: acesso total ao compartilhamento.
- `suporte_ti`: acesso apenas para leitura.
- `visitante_ti`: acesso negado.

Essa configuração permitiu controlar quais ações cada usuário poderia realizar dentro da pasta compartilhada.

## Criação dos usuários no sistema Linux

```bash
sudo useradd -M -s /sbin/nologin admin_ti
sudo useradd -M -s /sbin/nologin suporte_ti
sudo useradd -M -s /sbin/nologin visitante_ti
```
 <img width="946" height="580" alt="image" src="https://github.com/user-attachments/assets/3600a848-b465-455b-9914-1745a881b44f" />

 # Etapa 3 – Configuração do Samba e Compartilhamento

Após a instalação do Samba, os usuários foram adicionados ao sistema utilizando o comando `smbpasswd`. Em seguida, foi configurado o compartilhamento chamado `Arquivos_TI`, definindo quais usuários teriam acesso, leitura ou escrita na pasta compartilhada do servidor.

Essa etapa foi essencial para garantir a segurança e o controle de permissões dentro do ambiente de rede.

```bash
sudo smbpasswd -a admin_ti
sudo smbpasswd -a suporte_ti
sudo smbpasswd -a visitante_ti

Senha utilizada no laboratório:
123
```
 <img width="492" height="228" alt="image" src="https://github.com/user-attachments/assets/a387a4ca-89dd-43b8-b2d5-3fcb7a4b7d14" />

## Configuração do Compartilhamento no Samba

```ini
[Arquivos_TI]
path = /arquivos_ti
browseable = yes
valid users = admin_ti, suporte_ti
read list = suporte_ti
write list = admin_ti
````
 <img width="963" height="810" alt="image" src="https://github.com/user-attachments/assets/5831b714-db47-4681-8b12-c4bfe1551ab7" />
 <img width="952" height="937" alt="image" src="https://github.com/user-attachments/assets/a4bc8efb-a0f6-4d1e-93de-7158228e225f" />

# Etapa 4 – Configuração da Máquina Cliente

Na máquina cliente foram instaladas as ferramentas necessárias para realizar o mapeamento da rede, utilizando o pacote `cifs-utils`. Depois disso, foi criado o diretório `/mnt/rede_ti`, que serviu como ponto de montagem para acessar os arquivos compartilhados pelo servidor.

Com essa configuração, a máquina cliente conseguiu acessar os arquivos disponíveis na rede de forma organizada.

## Comandos executados na máquina cliente

```bash
sudo apt update
sudo apt install cifs-utils -y
sudo mkdir -p /mnt/rede_ti
````

 <img width="927" height="941" alt="image" src="https://github.com/user-attachments/assets/e35a84a4-d1e8-49d0-9baa-680206c6dec7" />

# Etapa 5 – Testes de Permissões dos Usuários

Nesta etapa foram realizados os testes práticos de acesso ao compartilhamento utilizando os três usuários criados anteriormente.

O usuário `admin_ti` conseguiu acessar, visualizar e criar arquivos normalmente, comprovando que possuía permissões de leitura e escrita.

O usuário `suporte_ti` conseguiu apenas visualizar os arquivos da pasta compartilhada, mas recebeu erro de permissão ao tentar criar novos arquivos, demonstrando que o acesso estava configurado somente para leitura.

Já o usuário `visitante_ti` não conseguiu acessar o compartilhamento, recebendo mensagem de erro de permissão, confirmando que o acesso havia sido bloqueado corretamente.

---

## Teste 1 – Perfil Administrador (Leitura e Escrita Permitidas)

### Comandos executados

```bash
sudo mount -t cifs -o username=admin_ti //IP_DO_SERVIDOR/Arquivos_TI /mnt/rede_ti

cd /mnt/rede_ti

echo "Teste Admin" | sudo tee relatorio.txt

cd ~

sudo umount /mnt/rede_ti
````

<img width="743" height="421" alt="image" src="https://github.com/user-attachments/assets/84562166-fdcf-489d-87bb-967d463c0d34" />
<img width="955" height="128" alt="image" src="https://github.com/user-attachments/assets/677702a0-4212-43fe-9f44-c51f7072f608" />

## Teste 2 – Perfil Suporte (Apenas Leitura Permitida)

### Comandos executados

```bash
sudo mount -t cifs -o username=suporte_ti //IP_DO_SERVIDOR/Arquivos_TI /mnt/rede_ti

ls -l /mnt/rede_ti

echo "Teste Suporte" | sudo tee /mnt/rede_ti/teste2.txt

sudo umount /mnt/rede_ti
```

O usuário `suporte_ti` conseguiu visualizar os arquivos presentes no compartilhamento, incluindo o arquivo `relatorio.txt`. Entretanto, ao tentar criar um novo arquivo, o sistema retornou erro de permissão, confirmando que o usuário possuía apenas acesso de leitura.

---

## Teste 3 – Perfil Visitante (Acesso Negado)

### Comando executado

```bash
sudo mount -t cifs -o username=visitante_ti //IP_DO_SERVIDOR/Arquivos_TI /mnt/rede_ti
```

O comando de montagem falhou e retornou erro de permissão, demonstrando que o usuário `visitante_ti` não possuía autorização para acessar o compartilhamento de rede.

<img width="964" height="117" alt="image" src="https://github.com/user-attachments/assets/79cfcdc4-a842-4aab-8005-e2d6450302ac" />
<img width="959" height="78" alt="image" src="https://github.com/user-attachments/assets/18f7e460-0195-4b4c-b884-157b19845751" />
<img width="255" height="103" alt="image" src="https://github.com/user-attachments/assets/5734e911-7f81-4213-958c-e580efae87a0" />
<img width="631" height="378" alt="image" src="https://github.com/user-attachments/assets/3a1cd05f-9643-4a66-a7e9-2188628b6cc3" />

# 6. Persistência de Dados e Automontagem (/etc/fstab)

O mapeamento realizado manualmente através do comando `mount` é temporário, sendo removido automaticamente após a reinicialização do sistema. Como parte do desafio de autonomia, foi realizada a configuração do arquivo `/etc/fstab` na máquina cliente.

Foi adicionada uma entrada nesse arquivo para que o Kernel execute a montagem automática do compartilhamento CIFS durante o processo de inicialização do sistema. Nessa configuração, foram utilizadas as credenciais do usuário administrador e definido o padrão de codificação `utf8`, garantindo a correta exibição de caracteres especiais e acentuações nos nomes dos arquivos.

<img width="759" height="369" alt="image" src="https://github.com/user-attachments/assets/6ef531b2-c6a6-4d56-932e-d7a8653e3a87" />


# Conclusão

A realização desta atividade foi muito importante para entender, na prática, como funciona a configuração de um servidor Linux utilizando o Samba para compartilhamento de arquivos em rede. Durante o desenvolvimento, foi possível aprender desde a criação de usuários e definição de permissões até os testes de acesso realizados entre a máquina cliente e o servidor.

Os testes mostraram claramente como cada perfil possui diferentes níveis de acesso, garantindo mais segurança e controle dentro do ambiente de rede. Além disso, a atividade ajudou a desenvolver conhecimentos essenciais sobre administração de servidores Linux, comunicação entre máquinas e gerenciamento de compartilhamentos.

Ao final, todos os objetivos propostos foram alcançados com sucesso, demonstrando o funcionamento correto das permissões configuradas e reforçando a importância de uma boa organização e segurança em redes de computadores.







