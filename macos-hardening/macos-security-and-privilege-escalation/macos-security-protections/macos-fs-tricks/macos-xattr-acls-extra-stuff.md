# macOS xattr-acls extra stuff

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}
```bash
rm -rf /tmp/test*
echo test >/tmp/test
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" /tmp/test
./get_acls test
ACL for test:
!#acl 1
group:ABCDEFAB-CDEF-ABCD-EFAB-CDEF0000000C:everyone:12:deny:write,writeattr,writeextattr,writesecurity,chown

ACL in hex: \x21\x23\x61\x63\x6c\x20\x31\x0a\x67\x72\x6f\x75\x70\x3a\x41\x42\x43\x44\x45\x46\x41\x42\x2d\x43\x44\x45\x46\x2d\x41\x42\x43\x44\x2d\x45\x46\x41\x42\x2d\x43\x44\x45\x46\x30\x30\x30\x30\x30\x30\x30\x43\x3a\x65\x76\x65\x72\x79\x6f\x6e\x65\x3a\x31\x32\x3a\x64\x65\x6e\x79\x3a\x77\x72\x69\x74\x65\x2c\x77\x72\x69\x74\x65\x61\x74\x74\x72\x2c\x77\x72\x69\x74\x65\x65\x78\x74\x61\x74\x74\x72\x2c\x77\x72\x69\x74\x65\x73\x65\x63\x75\x72\x69\x74\x79\x2c\x63\x68\x6f\x77\x6e\x0a
```
<details>

<summary>Código de get_acls</summary>
```c
// gcc -o get_acls get_acls
#include <stdio.h>
#include <stdlib.h>
#include <sys/acl.h>

int main(int argc, char *argv[]) {
if (argc != 2) {
fprintf(stderr, "Usage: %s <filepath>\n", argv[0]);
return 1;
}

const char *filepath = argv[1];
acl_t acl = acl_get_file(filepath, ACL_TYPE_EXTENDED);
if (acl == NULL) {
perror("acl_get_file");
return 1;
}

char *acl_text = acl_to_text(acl, NULL);
if (acl_text == NULL) {
perror("acl_to_text");
acl_free(acl);
return 1;
}

printf("ACL for %s:\n%s\n", filepath, acl_text);

// Convert acl_text to hexadecimal and print it
printf("ACL in hex: ");
for (char *c = acl_text; *c != '\0'; c++) {
printf("\\x%02x", (unsigned char)*c);
}
printf("\n");

acl_free(acl);
acl_free(acl_text);
return 0;
}
```
```markdown
<details>
<summary>Extra Stuff</summary>

## Extended Attributes (xattr)

No macOS, os atributos estendidos (xattr) são metadados que podem ser associados a arquivos e diretórios. Eles podem ser usados para armazenar informações adicionais sobre um arquivo, como a origem, permissões ou outros dados relevantes.

### Como visualizar atributos estendidos

Para visualizar os atributos estendidos de um arquivo, você pode usar o seguinte comando:

```bash
xattr -l <caminho_do_arquivo>
```

### Como adicionar atributos estendidos

Para adicionar um atributo estendido a um arquivo, use o comando:

```bash
xattr -w <nome_do_atributo> <valor> <caminho_do_arquivo>
```

### Como remover atributos estendidos

Para remover um atributo estendido, utilize:

```bash
xattr -d <nome_do_atributo> <caminho_do_arquivo>
```

## Listas de Controle de Acesso (ACLs)

As ACLs no macOS permitem um controle mais granular sobre as permissões de arquivos e diretórios. Elas podem ser usadas para definir permissões específicas para usuários ou grupos.

### Como visualizar ACLs

Para visualizar as ACLs de um arquivo ou diretório, use:

```bash
ls -le <caminho_do_arquivo>
```

### Como adicionar uma ACL

Para adicionar uma ACL, utilize o comando:

```bash
chmod +a "<usuário ou grupo> allow <permissão>" <caminho_do_arquivo>
```

### Como remover uma ACL

Para remover uma ACL, use:

```bash
chmod -a "<usuário ou grupo>" <caminho_do_arquivo>
```

</details>
```
```bash
# Lets add the xattr com.apple.xxx.xxxx with the acls
mkdir start
mkdir start/protected
./set_xattr start/protected
echo something > start/protected/something
```
<details>

<summary>Código de set_xattr</summary>
```c
// gcc -o set_xattr set_xattr.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/xattr.h>
#include <sys/acl.h>


void print_xattrs(const char *filepath) {
ssize_t buflen = listxattr(filepath, NULL, 0, XATTR_NOFOLLOW);
if (buflen < 0) {
perror("listxattr");
return;
}

char *buf = malloc(buflen);
if (buf == NULL) {
perror("malloc");
return;
}

buflen = listxattr(filepath, buf, buflen, XATTR_NOFOLLOW);
if (buflen < 0) {
perror("listxattr");
free(buf);
return;
}

printf("All current extended attributes for %s:\n", filepath);
for (char *name = buf; name < buf + buflen; name += strlen(name) + 1) {
printf("%s: ", name);
ssize_t valuelen = getxattr(filepath, name, NULL, 0, 0, XATTR_NOFOLLOW);
if (valuelen < 0) {
perror("getxattr");
continue;
}

char *value = malloc(valuelen + 1);
if (value == NULL) {
perror("malloc");
continue;
}

valuelen = getxattr(filepath, name, value, valuelen, 0, XATTR_NOFOLLOW);
if (valuelen < 0) {
perror("getxattr");
free(value);
continue;
}

value[valuelen] = '\0';  // Null-terminate the value
printf("%s\n", value);
free(value);
}

free(buf);
}


int main(int argc, char *argv[]) {
if (argc != 2) {
fprintf(stderr, "Usage: %s <filepath>\n", argv[0]);
return 1;
}

const char *hex = "\x21\x23\x61\x63\x6c\x20\x31\x0a\x67\x72\x6f\x75\x70\x3a\x41\x42\x43\x44\x45\x46\x41\x42\x2d\x43\x44\x45\x46\x2d\x41\x42\x43\x44\x2d\x45\x46\x41\x42\x2d\x43\x44\x45\x46\x30\x30\x30\x30\x30\x30\x30\x43\x3a\x65\x76\x65\x72\x79\x6f\x6e\x65\x3a\x31\x32\x3a\x64\x65\x6e\x79\x3a\x77\x72\x69\x74\x65\x2c\x77\x72\x69\x74\x65\x61\x74\x74\x72\x2c\x77\x72\x69\x74\x65\x65\x78\x74\x61\x74\x74\x72\x2c\x77\x72\x69\x74\x65\x73\x65\x63\x75\x72\x69\x74\x79\x2c\x63\x68\x6f\x77\x6e\x0a";
const char *filepath = argv[1];

int result = setxattr(filepath, "com.apple.xxx.xxxx", hex, strlen(hex), 0, 0);
if (result == 0) {
printf("Extended attribute set successfully.\n\n");
} else {
perror("setxattr");
return 1;
}

print_xattrs(filepath);

return 0;
}
```
</details>

{% code overflow="wrap" %}
```bash
# Create appledoublefile with the xattr entitlement
ditto -c -k start protected.zip
rm -rf start
# extract the files
unzip protected.zip
# Replace the name of the xattr here (if you put it before ditto would have destroyed it)
python3 -c "with open('._protected', 'rb+') as f: content = f.read().replace(b'com.apple.xxx.xxxx', b'com.apple.acl.text'); f.seek(0); f.write(content); f.truncate()"
# zip everything back together
rm -rf protected.zip
zip -r protected.zip protected ._protected
rm -rf protected
rm ._*
```
{% endcode %}
```bash
# Check if it worked
ditto -x -k --rsrc protected.zip .
xattr -l protected
```
{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Suporte ao HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}
