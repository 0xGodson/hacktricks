# macOS Keychain

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to **silnik wyszukiwania** zasilany **dark-web**, który oferuje **darmowe** funkcjonalności do sprawdzenia, czy firma lub jej klienci zostali **skompromentowani** przez **złośliwe oprogramowanie kradnące**.

Ich głównym celem jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik za **darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

***

## Główne Keychainy

* **Keychain Użytkownika** (`~/Library/Keychains/login.keycahin-db`), który służy do przechowywania **poświadczeń specyficznych dla użytkownika**, takich jak hasła aplikacji, hasła internetowe, certyfikaty generowane przez użytkownika, hasła sieciowe oraz klucze publiczne/prywatne generowane przez użytkownika.
* **Keychain Systemowy** (`/Library/Keychains/System.keychain`), który przechowuje **poświadczenia systemowe**, takie jak hasła WiFi, certyfikaty główne systemu, prywatne klucze systemowe oraz hasła aplikacji systemowych.

### Dostęp do Keychainu z Hasłem

Te pliki, mimo że nie mają wbudowanej ochrony i mogą być **pobrane**, są szyfrowane i wymagają **czystego hasła użytkownika do odszyfrowania**. Narzędzie takie jak [**Chainbreaker**](https://github.com/n0fate/chainbreaker) może być użyte do odszyfrowania.

## Ochrona Wpisów w Keychainie

### ACLs

Każdy wpis w keychainie jest regulowany przez **Listy Kontroli Dostępu (ACLs)**, które określają, kto może wykonywać różne działania na wpisie w keychainie, w tym:

* **ACLAuhtorizationExportClear**: Pozwala posiadaczowi uzyskać czysty tekst sekretu.
* **ACLAuhtorizationExportWrapped**: Pozwala posiadaczowi uzyskać czysty tekst zaszyfrowany innym podanym hasłem.
* **ACLAuhtorizationAny**: Pozwala posiadaczowi na wykonanie dowolnej akcji.

ACLs są dodatkowo uzupełnione o **listę zaufanych aplikacji**, które mogą wykonywać te działania bez wywoływania komunikatu. Może to być:

* **N`il`** (brak wymaganej autoryzacji, **wszyscy są zaufani**)
* **Pusta** lista (**nikt** nie jest zaufany)
* **Lista** konkretnych **aplikacji**.

Wpis może również zawierać klucz **`ACLAuthorizationPartitionID`,** który służy do identyfikacji **teamid, apple** i **cdhash.**

* Jeśli **teamid** jest określony, to aby **uzyskać dostęp do wartości wpisu** **bez** komunikatu, używana aplikacja musi mieć **to samo teamid**.
* Jeśli **apple** jest określony, to aplikacja musi być **podpisana** przez **Apple**.
* Jeśli **cdhash** jest wskazany, to **aplikacja** musi mieć konkretny **cdhash**.

### Tworzenie Wpisu w Keychainie

Gdy **nowy** **wpis** jest tworzony za pomocą **`Keychain Access.app`**, obowiązują następujące zasady:

* Wszystkie aplikacje mogą szyfrować.
* **Żadne aplikacje** nie mogą eksportować/odszyfrowywać (bez wywoływania komunikatu do użytkownika).
* Wszystkie aplikacje mogą zobaczyć kontrolę integralności.
* Żadne aplikacje nie mogą zmieniać ACLs.
* **partitionID** jest ustawiony na **`apple`**.

Gdy **aplikacja tworzy wpis w keychainie**, zasady są nieco inne:

* Wszystkie aplikacje mogą szyfrować.
* Tylko **tworząca aplikacja** (lub inne aplikacje wyraźnie dodane) mogą eksportować/odszyfrowywać (bez wywoływania komunikatu do użytkownika).
* Wszystkie aplikacje mogą zobaczyć kontrolę integralności.
* Żadne aplikacje nie mogą zmieniać ACLs.
* **partitionID** jest ustawiony na **`teamid:[teamID tutaj]`**.

## Uzyskiwanie Dostępu do Keychainu

### `security`
```bash
# List keychains
security list-keychains

# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S

# Dump specifically the user keychain
security dump-keychain ~/Library/Keychains/login.keychain-db
```
### APIs

{% hint style="success" %}
Enumeracja i zrzut **keychain** sekretów, które **nie wygenerują powiadomienia**, można wykonać za pomocą narzędzia [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Lista i uzyskanie **informacji** o każdym wpisie w keychain:

* API **`SecItemCopyMatching`** daje informacje o każdym wpisie i są pewne atrybuty, które można ustawić podczas jego używania:
* **`kSecReturnData`**: Jeśli prawda, spróbuje odszyfrować dane (ustaw na fałsz, aby uniknąć potencjalnych wyskakujących okienek)
* **`kSecReturnRef`**: Uzyskaj również odniesienie do elementu keychain (ustaw na prawda, jeśli później zobaczysz, że możesz odszyfrować bez wyskakującego okienka)
* **`kSecReturnAttributes`**: Uzyskaj metadane o wpisach
* **`kSecMatchLimit`**: Ile wyników zwrócić
* **`kSecClass`**: Jakiego rodzaju wpis w keychain

Uzyskaj **ACL** każdego wpisu:

* Za pomocą API **`SecAccessCopyACLList`** możesz uzyskać **ACL dla elementu keychain**, a zwróci to listę ACL (takich jak `ACLAuhtorizationExportClear` i inne wcześniej wspomniane), gdzie każda lista ma:
* Opis
* **Lista Zaufanych Aplikacji**. To może być:
* Aplikacja: /Applications/Slack.app
* Binarny: /usr/libexec/airportd
* Grupa: group://AirPort

Eksportuj dane:

* API **`SecKeychainItemCopyContent`** uzyskuje tekst jawny
* API **`SecItemExport`** eksportuje klucze i certyfikaty, ale może być konieczne ustawienie haseł do eksportu zawartości w formie zaszyfrowanej

A oto **wymagania**, aby móc **eksportować sekret bez powiadomienia**:

* Jeśli **1+ zaufane** aplikacje wymienione:
* Potrzebne odpowiednie **autoryzacje** (**`Nil`**, lub być **częścią** dozwolonej listy aplikacji w autoryzacji do uzyskania dostępu do sekretnej informacji)
* Potrzebny podpis kodu, aby pasował do **PartitionID**
* Potrzebny podpis kodu, aby pasował do jednego **zaufanego programu** (lub być członkiem odpowiedniej grupy KeychainAccessGroup)
* Jeśli **wszystkie aplikacje zaufane**:
* Potrzebne odpowiednie **autoryzacje**
* Potrzebny podpis kodu, aby pasował do **PartitionID**
* Jeśli **brak PartitionID**, to nie jest potrzebne

{% hint style="danger" %}
Dlatego, jeśli jest **1 aplikacja wymieniona**, musisz **wstrzyknąć kod w tę aplikację**.

Jeśli **apple** jest wskazane w **partitionID**, możesz uzyskać do niego dostęp za pomocą **`osascript`**, więc wszystko, co ufa wszystkim aplikacjom z apple w partitionID. **`Python`** również może być użyty do tego.
{% endhint %}

### Dwa dodatkowe atrybuty

* **Niewidoczny**: To booleanowy znacznik do **ukrycia** wpisu w aplikacji **UI** Keychain
* **Ogólny**: Służy do przechowywania **metadanych** (więc NIE JEST ZASZYFROWANY)
* Microsoft przechowywał w formie jawnej wszystkie tokeny odświeżania do uzyskania dostępu do wrażliwego punktu końcowego.

## Referencje

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to **wyszukiwarka** zasilana **dark-web**, która oferuje **darmowe** funkcjonalności do sprawdzenia, czy firma lub jej klienci zostali **skompromentowani** przez **złośliwe oprogramowanie kradnące**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik za **darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}
