# macOS MACF

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basic Information

**MACF** означає **Систему обов'язкового контролю доступу**, яка є системою безпеки, вбудованою в операційну систему для захисту вашого комп'ютера. Вона працює, встановлюючи **суворі правила щодо того, хто або що може отримати доступ до певних частин системи**, таких як файли, програми та системні ресурси. Застосовуючи ці правила автоматично, MACF забезпечує, що лише авторизовані користувачі та процеси можуть виконувати певні дії, зменшуючи ризик несанкціонованого доступу або шкідливих дій.

Зверніть увагу, що MACF насправді не приймає жодних рішень, оскільки просто **перехоплює** дії, залишаючи рішення **модулям політики** (розширенням ядра), які вона викликає, таким як `AppleMobileFileIntegrity.kext`, `Quarantine.kext`, `Sandbox.kext`, `TMSafetyNet.kext` та `mcxalr.kext`.

### Flow

1. Процес виконує syscall/mach trap
2. Відповідна функція викликається всередині ядра
3. Функція викликає MACF
4. MACF перевіряє модулі політики, які запитували підключення цієї функції у своїй політиці
5. MACF викликає відповідні політики
6. Політики вказують, чи дозволяють або забороняють дію

{% hint style="danger" %}
Apple є єдиною компанією, яка може використовувати KPI MAC Framework.
{% endhint %}

### Labels

MACF використовує **мітки**, які потім політики перевіряють, чи повинні вони надати доступ чи ні. Код оголошення структури міток можна [знайти тут](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/_label.h), яка потім використовується всередині **`struct ucred`** [**тут**](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/sys/ucred.h#L86) в частині **`cr_label`**. Мітка містить прапорці та кількість **слотів**, які можуть використовуватися **політиками MACF для виділення вказівників**. Наприклад, Sanbox вказуватиме на профіль контейнера.

## MACF Policies

Політика MACF визначає **правила та умови, які застосовуються до певних операцій ядра**.&#x20;

Розширення ядра може налаштувати структуру `mac_policy_conf`, а потім зареєструвати її, викликавши `mac_policy_register`. З [тут](https://opensource.apple.com/source/xnu/xnu-2050.18.24/security/mac_policy.h.auto.html):
```c
#define mpc_t	struct mac_policy_conf *

/**
@brief Mac policy configuration

This structure specifies the configuration information for a
MAC policy module.  A policy module developer must supply
a short unique policy name, a more descriptive full name, a list of label
namespaces and count, a pointer to the registered enty point operations,
any load time flags, and optionally, a pointer to a label slot identifier.

The Framework will update the runtime flags (mpc_runtime_flags) to
indicate that the module has been registered.

If the label slot identifier (mpc_field_off) is NULL, the Framework
will not provide label storage for the policy.  Otherwise, the
Framework will store the label location (slot) in this field.

The mpc_list field is used by the Framework and should not be
modified by policies.
*/
/* XXX - reorder these for better aligment on 64bit platforms */
struct mac_policy_conf {
const char		*mpc_name;		/** policy name */
const char		*mpc_fullname;		/** full name */
const char		**mpc_labelnames;	/** managed label namespaces */
unsigned int		 mpc_labelname_count;	/** number of managed label namespaces */
struct mac_policy_ops	*mpc_ops;		/** operation vector */
int			 mpc_loadtime_flags;	/** load time flags */
int			*mpc_field_off;		/** label slot */
int			 mpc_runtime_flags;	/** run time flags */
mpc_t			 mpc_list;		/** List reference */
void			*mpc_data;		/** module data */
};
```
Легко ідентифікувати розширення ядра, які налаштовують ці політики, перевіряючи виклики до `mac_policy_register`. Більше того, перевіряючи дизасемблювання розширення, також можна знайти використану структуру `mac_policy_conf`.

Зверніть увагу, що політики MACF можуть бути зареєстровані та скасовані також **динамічно**.

Одним з основних полів `mac_policy_conf` є **`mpc_ops`**. Це поле вказує, які операції цікавлять політику. Зверніть увагу, що їх сотні, тому можливо обнулити всі з них, а потім вибрати лише ті, які цікавлять політику. З [тут](https://opensource.apple.com/source/xnu/xnu-2050.18.24/security/mac\_policy.h.auto.html):
```c
struct mac_policy_ops {
mpo_audit_check_postselect_t		*mpo_audit_check_postselect;
mpo_audit_check_preselect_t		*mpo_audit_check_preselect;
mpo_bpfdesc_label_associate_t		*mpo_bpfdesc_label_associate;
mpo_bpfdesc_label_destroy_t		*mpo_bpfdesc_label_destroy;
mpo_bpfdesc_label_init_t		*mpo_bpfdesc_label_init;
mpo_bpfdesc_check_receive_t		*mpo_bpfdesc_check_receive;
mpo_cred_check_label_update_execve_t	*mpo_cred_check_label_update_execve;
mpo_cred_check_label_update_t		*mpo_cred_check_label_update;
[...]
```
```markdown
Практично всі хуки будуть викликані MACF, коли одна з цих операцій буде перехоплена. Однак, **`mpo_policy_*`** хуки є винятком, оскільки `mpo_hook_policy_init()` є зворотним викликом, який викликається під час реєстрації (тобто після `mac_policy_register()`), а `mpo_hook_policy_initbsd()` викликається під час пізньої реєстрації, коли підсистема BSD була правильно ініціалізована.

Більше того, **`mpo_policy_syscall`** хук може бути зареєстрований будь-яким kext для відкриття приватного **ioctl** стилю виклику **інтерфейсу**. Тоді клієнт користувача зможе викликати `mac_syscall` (#381), вказуючи в якості параметрів **ім'я політики** з цілим **кодом** та необов'язковими **аргументами**.\
Наприклад, **`Sandbox.kext`** використовує це дуже часто.

Перевіряючи **`__DATA.__const*`** kext, можна ідентифікувати структуру `mac_policy_ops`, яка використовується під час реєстрації політики. Це можливо знайти, оскільки її вказівник знаходиться на зсуві всередині `mpo_policy_conf`, а також через кількість NULL вказівників, які будуть у цій області.

Крім того, також можливо отримати список kext, які налаштували політику, скидаючи з пам'яті структуру **`_mac_policy_list`**, яка оновлюється з кожною зареєстрованою політикою.

## Ініціалізація MACF

MACF ініціалізується дуже швидко. Він налаштовується в `bootstrap_thread` XNU: після `ipc_bootstrap` викликається `mac_policy_init()`, яка ініціалізує `mac_policy_list`, а через кілька моментів викликається `mac_policy_initmach()`. Серед іншого, ця функція отримає всі Apple kext з ключем `AppleSecurityExtension` у їх Info.plist, такі як `ALF.kext`, `AppleMobileFileIntegrity.kext`, `Quarantine.kext`, `Sandbox.kext` та `TMSafetyNet.kext` і завантажить їх.

## Виклики MACF

Звичайно, можна знайти виклики до MACF, визначені в коді, такі як: **`#if CONFIG_MAC`** умовні блоки. Більше того, всередині цих блоків можна знайти виклики до `mac_proc_check*`, які викликають MACF для **перевірки дозволів** на виконання певних дій. Крім того, формат викликів MACF є: **`mac_<object>_<opType>_opName`**.

Об'єкт є одним з наступних: `bpfdesc`, `cred`, `file`, `proc`, `vnode`, `mount`, `devfs`, `ifnet`, `inpcb`, `mbuf`, `ipq`, `pipe`, `sysv[msg/msq/shm/sem]`, `posix[shm/sem]`, `socket`, `kext`.\
`opType` зазвичай є check, який буде використовуватися для дозволу або заборони дії. Однак також можливо знайти `notify`, що дозволить kext реагувати на дану дію.

Ви можете знайти приклад у [https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/kern/kern\_mman.c#L621](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/kern/kern\_mman.c#L621):

<pre class="language-c"><code class="lang-c">int
mmap(proc_t p, struct mmap_args *uap, user_addr_t *retval)
{
[...]
#if CONFIG_MACF
<strong>			error = mac_file_check_mmap(vfs_context_ucred(ctx),
</strong>			    fp->fp_glob, prot, flags, file_pos + pageoff,
&#x26;maxprot);
if (error) {
(void)vnode_put(vp);
goto bad;
}
#endif /* MAC */
[...]
</code></pre>

Тоді можливо знайти код `mac_file_check_mmap` у [https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac\_file.c#L174](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac\_file.c#L174)
```
```c
mac_file_check_mmap(struct ucred *cred, struct fileglob *fg, int prot,
int flags, uint64_t offset, int *maxprot)
{
int error;
int maxp;

maxp = *maxprot;
MAC_CHECK(file_check_mmap, cred, fg, NULL, prot, flags, offset, &maxp);
if ((maxp | *maxprot) != *maxprot) {
panic("file_check_mmap increased max protections");
}
*maxprot = maxp;
return error;
}
```
Який викликає макрос `MAC_CHECK`, код якого можна знайти за посиланням [https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac\_internal.h#L261](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac\_internal.h#L261)
```c
/*
* MAC_CHECK performs the designated check by walking the policy
* module list and checking with each as to how it feels about the
* request.  Note that it returns its value via 'error' in the scope
* of the caller.
*/
#define MAC_CHECK(check, args...) do {                              \
error = 0;                                                      \
MAC_POLICY_ITERATE({                                            \
if (mpc->mpc_ops->mpo_ ## check != NULL) {              \
DTRACE_MACF3(mac__call__ ## check, void *, mpc, int, error, int, MAC_ITERATE_CHECK); \
int __step_err = mpc->mpc_ops->mpo_ ## check (args); \
DTRACE_MACF2(mac__rslt__ ## check, void *, mpc, int, __step_err); \
error = mac_error_select(__step_err, error);         \
}                                                           \
});                                                             \
} while (0)
```
Який пройде через всі зареєстровані політики mac, викликаючи їх функції та зберігаючи вихідні дані в змінній помилки, яка може бути перевизначена лише `mac_error_select` кодами успіху, тому якщо будь-яка перевірка не пройде, вся перевірка зазнає невдачі, і дія не буде дозволена.

{% hint style="success" %}
Однак пам'ятайте, що не всі виклики MACF використовуються лише для відмови в діях. Наприклад, `mac_priv_grant` викликає макрос [**MAC\_GRANT**](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac\_internal.h#L274), який надасть запитувану привілегію, якщо будь-яка політика відповість 0:
```c
/*
* MAC_GRANT performs the designated check by walking the policy
* module list and checking with each as to how it feels about the
* request.  Unlike MAC_CHECK, it grants if any policies return '0',
* and otherwise returns EPERM.  Note that it returns its value via
* 'error' in the scope of the caller.
*/
#define MAC_GRANT(check, args...) do {                              \
error = EPERM;                                                  \
MAC_POLICY_ITERATE({                                            \
if (mpc->mpc_ops->mpo_ ## check != NULL) {                  \
DTRACE_MACF3(mac__call__ ## check, void *, mpc, int, error, int, MAC_ITERATE_GRANT); \
int __step_res = mpc->mpc_ops->mpo_ ## check (args); \
if (__step_res == 0) {                              \
error = 0;                                  \
}                                                   \
DTRACE_MACF2(mac__rslt__ ## check, void *, mpc, int, __step_res); \
}                                                           \
});                                                             \
} while (0)
```
{% endhint %}

### priv\_check & priv\_grant

Ці виклики призначені для перевірки та надання (десятків) **привілеїв**, визначених у [**bsd/sys/priv.h**](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/sys/priv.h).\
Деякий код ядра викликатиме `priv_check_cred()` з [**bsd/kern/kern\_priv.c**](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/kern/kern\_priv.c) з KAuth обліковими даними процесу та одним з кодів привілеїв, який викликатиме `mac_priv_check`, щоб перевірити, чи будь-яка політика **забороняє** надання привілею, а потім викликатиме `mac_priv_grant`, щоб перевірити, чи будь-яка політика надає `привілей`.

### proc\_check\_syscall\_unix

Цей хук дозволяє перехоплювати всі системні виклики. У `bsd/dev/[i386|arm]/systemcalls.c` можна побачити оголошену функцію [`unix_syscall`](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/bsd/dev/arm/systemcalls.c#L160C1-L167C25), яка містить цей код:
```c
#if CONFIG_MACF
if (__improbable(proc_syscall_filter_mask(proc) != NULL && !bitstr_test(proc_syscall_filter_mask(proc), syscode))) {
error = mac_proc_check_syscall_unix(proc, syscode);
if (error) {
goto skip_syscall;
}
}
#endif /* CONFIG_MACF */
```
Який перевірить у викликаючому процесі **бітову маску**, чи слід поточному системному виклику викликати `mac_proc_check_syscall_unix`. Це пов'язано з тим, що системні виклики викликаються так часто, що цікаво уникати виклику `mac_proc_check_syscall_unix` щоразу.

Зверніть увагу, що функція `proc_set_syscall_filter_mask()`, яка встановлює бітову маску системних викликів у процесі, викликається Sandbox для встановлення масок на пісочницях.

## Відкриті системні виклики MACF

Можливо взаємодіяти з MACF через деякі системні виклики, визначені в [security/mac.h](https://github.com/apple-oss-distributions/xnu/blob/94d3b452840153a99b38a3a9659680b2a006908e/security/mac.h#L151):
```c
/*
* Extended non-POSIX.1e interfaces that offer additional services
* available from the userland and kernel MAC frameworks.
*/
#ifdef __APPLE_API_PRIVATE
__BEGIN_DECLS
int      __mac_execve(char *fname, char **argv, char **envv, mac_t _label);
int      __mac_get_fd(int _fd, mac_t _label);
int      __mac_get_file(const char *_path, mac_t _label);
int      __mac_get_link(const char *_path, mac_t _label);
int      __mac_get_pid(pid_t _pid, mac_t _label);
int      __mac_get_proc(mac_t _label);
int      __mac_set_fd(int _fildes, const mac_t _label);
int      __mac_set_file(const char *_path, mac_t _label);
int      __mac_set_link(const char *_path, mac_t _label);
int      __mac_mount(const char *type, const char *path, int flags, void *data,
struct mac *label);
int      __mac_get_mount(const char *path, struct mac *label);
int      __mac_set_proc(const mac_t _label);
int      __mac_syscall(const char *_policyname, int _call, void *_arg);
__END_DECLS
#endif /*__APPLE_API_PRIVATE*/
```
## References

* [**\*OS Internals Volume III**](https://newosxbook.com/home.html)

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
