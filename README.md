# LAB3


## Настройка сетевой инфраструктуры в AWS

**Выполнил:** Andrei Boico
**Группа:** IA-240
**Год:** 2026
**Регион:** eu-central-1 (Frankfurt)

## Цель работы

Изучить создание и настройку виртуальной сети в AWS, работу с VPC, подсетями, маршрутизацией, Security Groups, NAT Gateway и EC2.

---

## 1. Создание VPC и подсетей

Была создана VPC:

* **Name:** `timecapsule-dev-vpc`
* **CIDR:** `10.4.0.0/16`

Созданы четыре подсети:

| Subnet                      | AZ            | CIDR           |
| --------------------------- | ------------- | -------------- |
| `timecapsule-dev-public-a`  | eu-central-1a | `10.4.0.0/24`  |
| `timecapsule-dev-public-b`  | eu-central-1b | `10.4.1.0/24`  |
| `timecapsule-dev-private-a` | eu-central-1a | `10.4.10.0/24` |
| `timecapsule-dev-private-b` | eu-central-1b | `10.4.11.0/24` |

Для ресурсов были установлены необходимые теги проекта.


<img width="1919" height="596" alt="Снимок экрана 2026-10-05 200024" src="https://github.com/user-attachments/assets/0019521e-ca5e-48b9-8239-1f9c745e6084" />

---

## 2. Internet Gateway и маршрутизация

Был создан и подключён Internet Gateway:

`timecapsule-dev-igw`

Созданы таблицы маршрутизации:

* `timecapsule-dev-public-rt`
* `timecapsule-dev-private-rt`

Для публичной таблицы добавлен маршрут:

```text
0.0.0.0/0 → Internet Gateway
```

Публичная таблица связана с `public-a` и `public-b`, а приватная — с `private-a` и `private-b`.

<img width="1369" height="1115" alt="Снимок экрана 2026-10-05 202123" src="https://github.com/user-attachments/assets/81cb3cbb-500a-4720-9a06-f37668d49e68" />
<img width="1122" height="1111" alt="Снимок экрана 2026-10-05 202113" src="https://github.com/user-attachments/assets/f25cb0e3-da85-4c4c-9262-59311d1077ad" />


## 3. Security Groups и EC2

Созданы Security Groups:

* `timecapsule-dev-bastion-sg`
* `timecapsule-dev-web-sg`
* `timecapsule-dev-db-sg`

Основные правила:

| Security Group | Разрешённый доступ                        |
| -------------- | ----------------------------------------- |
| Bastion        | SSH 22 от My IP                           |
| Web            | HTTP 80 из Интернета, SSH 22 от Bastion   |
| DB             | PostgreSQL 5432 от Web, SSH 22 от Bastion |

Созданы три EC2-инстанса:

* `timecapsule-dev-bastion`
* `timecapsule-dev-web`
* `timecapsule-dev-db`

Bastion и Web находятся в публичных подсетях, DB — в приватной подсети без публичного IP.

<img width="932" height="933" alt="Снимок экрана 2026-10-05 205211" src="https://github.com/user-attachments/assets/4ac811a5-e2fc-4ecd-8a38-3aed519bb19b" />


## 4. SSH и Web-сервер

Для подключения к серверам был настроен SSH Config.

Web и DB доступны через Bastion с использованием `ProxyJump`.

На Web-сервере был установлен Nginx:

```bash
sudo dnf -y install nginx
sudo systemctl enable --now nginx
```

После запуска Nginx Web-сервер стал доступен по HTTP.
<img width="751" height="380" alt="Снимок экрана 2026-10-05 210717" src="https://github.com/user-attachments/assets/2e03ebf8-a8ca-410c-aefc-3ff447cb7ab4" />


## 5. NAT Gateway и PostgreSQL

Для временного доступа DB-сервера в Интернет был создан NAT Gateway:

`timecapsule-dev-nat-a`

В private route table временно был добавлен маршрут:

```text
0.0.0.0/0 → NAT Gateway
```

На DB-сервере был установлен PostgreSQL 16.

Созданы:

* Database: `lab3`
* User: `student`

PostgreSQL был настроен для сетевых подключений.

После выполнения необходимых операций NAT Gateway был удалён.
<img width="885" height="842" alt="Снимок экрана 2026-10-05 210817" src="https://github.com/user-attachments/assets/0e0ae2d9-5564-4038-84d2-4f7b8143bae3" />

---

## 6. Проверка Web → DB и Network ACL

С Web-сервера было проверено подключение к PostgreSQL:

```bash
psql -h 10.4.10.141 -U student -d lab3
```

Подключение было успешно установлено.

Также был проверен порт PostgreSQL:

```bash
nc -zv 10.4.10.141 5432
```

Web имел доступ к DB, а подключение к PostgreSQL со стороны Bastion было заблокировано Security Group.

Дополнительно был проведён эксперимент с Network ACL. Временное правило `Deny` для собственного IP заблокировало SSH-доступ. После удаления правила доступ был восстановлен.

**Screenshots:**

* `[Вставить screenshot успешного подключения Web → DB]`
* `[Вставить screenshot проверки порта 5432]`
* `[Вставить screenshot Network ACL]`

---

## 7. Tag Editor и завершение работы

В AWS Tag Editor был выполнен поиск ресурсов по тегу:

```text
Project = timecapsule
```

В результате были найдены ресурсы созданной инфраструктуры.

**Screenshot №10:**
`[Вставить screenshot Tag Editor]`

После завершения работы были остановлены три EC2-инстанса:

* `timecapsule-dev-bastion`
* `timecapsule-dev-web`
* `timecapsule-dev-db`

<img width="652" height="410" alt="Снимок экрана 2026-10-05 211907" src="https://github.com/user-attachments/assets/757f0d7f-3b93-464f-8686-18172accb645" />

---

## Вывод

В ходе лабораторной работы была создана виртуальная сеть AWS с использованием VPC, публичных и приватных подсетей, Internet Gateway, NAT Gateway и таблиц маршрутизации.

Были настроены Security Groups, Bastion Host, Web-сервер с Nginx и приватный сервер PostgreSQL. Было проверено взаимодействие между Web и DB, а также работа Network ACL.

В результате были изучены основные принципы создания и защиты сетевой инфраструктуры в AWS.
