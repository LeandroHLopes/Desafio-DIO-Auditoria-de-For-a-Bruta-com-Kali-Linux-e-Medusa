# Desafio-DIO-Auditoria-de-For-a-Bruta-com-Kali-Linux-e-Medusa

Introdução e Objetivos
Este projeto prático faz parte do desafio de segurança ofensiva da Digital Innovation One (DIO), focado em compreender e simular ataques de força bruta em um ambiente controlado. A simulação foi realizada utilizando o sistema operacional Kali Linux e a ferramenta Medusa, tendo como alvo serviços vulneráveis da máquina virtual Metasploitable 2 e o aplicativo DVWA (Damn Vulnerable Web Application).
O principal objetivo deste desafio é exercitar as seguintes competências:
•	Configuração de Ambiente: Montagem de um laboratório de testes isolado (Rede Host-only).
•	Ataque e Exploração: Utilização de ferramentas como Medusa para testar a resistência de credenciais em diferentes protocolos (FTP, SMB, HTTP).
•	Análise de Vulnerabilidade: Identificação de falhas de autenticação.
•	Mitigação: Proposição de medidas de defesa e boas práticas de segurança.
•	Documentação: Registro claro e estruturado de todo o processo técnico.

 Configuração do Ambiente de Laboratório
Para garantir um ambiente de testes seguro e isolado, foram utilizadas duas máquinas virtuais interligadas por uma rede interna.
1. Máquinas Virtuais (VMs)
Função	Sistema Operacional/Aplicação	Endereço IP (Exemplo)	Usuário Padrão
Atacante	Kali Linux	192.168.56.101	root / kali
Alvo	Metasploitable 2 (Inclui DVWA)	192.168.56.102	msfadmin / msfadmin
2. Configuração de Rede
Ambas as VMs foram configuradas no VirtualBox utilizando o modo Rede Interna (Host-only). Essa configuração assegura que o tráfego de rede gerado pelos testes de ataque permaneça isolado, sem afetar a rede local (doméstica/corporativa).
•	Teste de Conectividade: Foi realizado um teste de ping a partir do Kali Linux para o Metasploitable 2 para confirmar que a comunicação entre as duas máquinas estava funcional.

 Execução dos Ataques de Força Bruta
Os testes de força bruta foram realizados utilizando o Medusa, uma ferramenta rápida, modular e paralela, ideal para ataques de login.
1. Cenário: Força Bruta em Serviço FTP
•	Objetivo: Tentar obter credenciais válidas para o serviço FTP do Metasploitable 2, que utiliza o vsftpd (Versão 2.3.4, conhecida por vulnerabilidades).
•	Wordlist Utilizada: Uma wordlist simples contida em wordlists/ftp_passwords.txt, focada em credenciais fracas e padrão.
•	Comando Medusa:
Bash
medusa -H 192.168.56.102 -u msfadmin -P /home/kali/Desktop/wordlists/ftp_passwords.txt -M ftp
•	Resultado:
o	Credencial Encontrada: msfadmin:msfadmin
o	Tempo de Execução: 14.5 segundos
•	Evidência: Link para a captura de tela do Medusa identificando a senha no /images/ftp_bruteforce.png.
2. Cenário: Password Spraying em Serviço SMB
•	Objetivo: Realizar um password spraying (testar uma senha comum contra uma grande lista de usuários) no serviço SMB.
•	Enumeração de Usuários (Etapa Prévia): Utilizou-se o enum4linux para listar usuários válidos. A lista de usuários foi salva em wordlists/smb_users.txt (incluindo msfadmin, user, postgres).
•	Senha Comum Testada: password
•	Comando Medusa:
Bash
medusa -H 192.168.56.102 -U /home/kali/Desktop/wordlists/smb_users.txt -p password -M smb
•	Resultado:
o	Credenciais Encontradas: user:password e msfadmin:password (em testes com o Metasploitable 2, a senha msfadmin muitas vezes é msfadmin, mas a senha password também é comum para outros usuários).
•	Evidência: Link para a captura de tela do Medusa identificando a senha no /images/smb_passwordspray.png.
3. Cenário: Automação de Login Web (DVWA)
•	Objetivo: Simular o ataque de força bruta no formulário de login do DVWA (Damn Vulnerable Web Application) na configuração de segurança Low (Baixa), que não possui token Anti-CSRF.
•	Ferramenta Utilizada: Hydra (usado por ser mais eficiente no protocolo HTTP POST para formulários web).
•	Método: Foi capturada a requisição HTTP de login para identificar os parâmetros corretos (username, password e o botão Login=Login).
•	Comando de Força Bruta:
Bash
hydra -l admin -P /home/kali/Desktop/wordlists/dvwa_passwords.txt 192.168.56.102 http-post-form "/vulnerabilities/brute/:username=^user^&password=^pass^&Login=Login:Login failed"
•	Resultado:
o	Credencial Encontrada: admin:password
•	Evidência: Link para a captura de tela do sucesso da automação no /images/dvwa_bruteforce.png.

 Recomendações de Mitigação de Ataques de Força Bruta
A principal lição aprendida é que a força bruta só é eficaz contra sistemas mal configurados. A seguir, estão as recomendações de segurança (mitigação) para prevenir ou dificultar esses ataques nos serviços testados:
Serviço	Vulnerabilidade Observada	Medidas de Mitigação (Prevenção)
Geral	Credenciais fracas e padrão.	Política de Senhas Fortes: Exigir senhas longas, complexas e exclusivas. Desencorajar e auditar o uso de senhas comuns.
FTP/SMB	Ausência de limitação de tentativas.	Bloqueio de Contas: Implementar mecanismos que bloqueiem ou atrasem o login (e.g., Rate Limiting ou Throttling) após X tentativas falhas em um curto período.
Web (DVWA)	Falta de proteção contra automação.	CAPTCHA e Tokens: Implementar CAPTCHA após falhas e usar Tokens Anti-CSRF variáveis para dificultar a automação de requisições.
Geral	Ausência de monitoramento.	Monitoramento e Alerta: Configurar sistemas de Intrusion Detection/Prevention System (IDS/IPS) e logs de autenticação para alertar a equipe de segurança sobre padrões de login incomuns (como muitas falhas em sequência).
Geral	Serviços desnecessários abertos.	Princípio do Privilégio Mínimo: Desativar todos os serviços (FTP, SMB, etc.) que não são estritamente necessários no servidor.
Geral	Tráfego em texto simples.	Criptografia: Garantir que a comunicação use protocolos criptografados (ex: SFTP em vez de FTP, HTTPS em vez de HTTP).

 Conclusão e Aprendizados
O desafio proporcionou uma visão prática da eficácia e da facilidade com que ferramentas de código aberto como o Medusa e o Hydra podem ser usadas para testar a segurança de sistemas. Fica evidente que a implementação de senhas fortes e o uso de rate limiting são as barreiras mais imediatas e eficazes contra ataques de força bruta.
A documentação detalhada, além de ser um requisito, reforça o aprendizado ao obrigar a análise e a estruturação das etapas do teste de penetração.
🔗 Recursos do Projeto
Arquivo/Pasta	Descrição
README.md	Este arquivo, detalhando o setup e os resultados.
images/	Capturas de tela (evidências) dos testes realizados.
wordlists/	Arquivos .txt contendo as wordlists utilizadas em cada cenário.
config_virtualbox.txt	Arquivo opcional com detalhes da configuração de rede do VirtualBox.
