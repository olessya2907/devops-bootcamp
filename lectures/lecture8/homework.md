# Домашнее задание — Лекция 8

## Задача

Создать Terraform проект с двумя окружениями — **dev** и **prod** — из одного модуля.
Это стандартный паттерн в реальных командах: один и тот же код, разные параметры.
После `terraform apply` у тебя будет два работающих сервера с разными страницами.

**Ожидаемый результат:** `terraform plan` показывает `4 to add` (2 EC2 + 2 Security Group),
`terraform output` отдаёт два разных URL, оба открываются в браузере.

---

## Шаг 1 — Создать структуру проекта

Создай новую папку **вне репо** (например, рядом с ним):

```powershell
# Windows:
mkdir C:\Tools\Terraform\terraform-hw8
cd C:\Tools\Terraform\terraform-hw8
```

```bash
# Mac/Linux:
mkdir ~/terraform-hw8
cd ~/terraform-hw8
```

Скопируй модуль из репо:

```powershell
# Windows:
xcopy /E /I ПУТЬ_К_РЕПО\lectures\lecture8\modules modules
```

```bash
# Mac/Linux:
cp -r ПУТЬ_К_РЕПО/lectures/lecture8/modules ./modules
```

---

## Шаг 2 — Добавить переменную env_label в модуль

Открой `modules/ec2/variables.tf` и добавь новую переменную:

```hcl
variable "env_label" {
  description = "Environment label shown on the webpage (e.g. DEV, PROD)"
  type        = string
  default     = "DEV"
}
```

Открой `modules/ec2/main.tf` и замени строку с `echo` в user_data:

```hcl
# Было:
echo "<h1>Hello from Terraform!</h1><p>Deployed with IaC — Lecture 8</p>" \
  > /usr/share/nginx/html/index.html

# Стало:
echo "<h1>${var.env_label}</h1><p>Deployed with Terraform IaC</p>" \
  > /usr/share/nginx/html/index.html
```

---

## Шаг 3 — Написать main.tf

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-YOUR_ACCOUNT_ID"
    key            = "hw8/terraform.tfstate"
    region         = "us-west-2"
    profile        = "YOUR_AWS_PROFILE"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

provider "aws" {
  region  = "us-west-2"
  profile = "YOUR_AWS_PROFILE"
}

module "web_dev" {
  source        = "./modules/ec2"
  key_name      = var.key_name
  instance_type = "t2.micro"
  name_tag      = "myapp-dev"
  env_label     = "DEV ENVIRONMENT"
}

module "web_prod" {
  source        = "./modules/ec2"
  key_name      = var.key_name
  instance_type = "t2.micro"
  name_tag      = "myapp-prod"
  env_label     = "PROD ENVIRONMENT"
}
```

---

## Шаг 4 — Написать variables.tf и outputs.tf

**variables.tf:**

```hcl
variable "key_name" {
  description = "SSH key pair name"
  type        = string
}
```

**outputs.tf:**

```hcl
output "dev_url" {
  description = "DEV environment URL"
  value       = "http://${module.web_dev.public_ip}"
}

output "prod_url" {
  description = "PROD environment URL"
  value       = "http://${module.web_prod.public_ip}"
}
```

---

## Шаг 5 — Создать terraform.tfvars

```hcl
key_name = "имя-твоего-ключа"
```

---

## Шаг 6 — Запуск

```bash
terraform init
terraform plan
```

✅ Проверь: в плане должно быть `Plan: 4 to add` (2 Security Group + 2 EC2)

```bash
terraform apply -auto-approve
```

---

## ✅ Проверь себя

> Подожди 2–3 минуты после apply.

```bash
terraform output
```

Ожидаемый вывод:
```
dev_url  = "http://1.2.3.4"
prod_url = "http://5.6.7.8"
```

Открой **оба URL** в браузере:
- Первый показывает: **DEV ENVIRONMENT**
- Второй показывает: **PROD ENVIRONMENT**

Посмотри в AWS Console → EC2: должно быть два инстанса — `myapp-dev` и `myapp-prod`.

---

## Шаг 7 — Уничтожить ресурсы (обязательно!)

```bash
terraform destroy -auto-approve
```

Ожидаемый результат: `Destroy complete! Resources: 4 destroyed.`

---

## Поделись результатом в Slack

- Скриншот: `terraform apply` — `Resources: 4 added`
- Скриншот: два браузера открыты рядом — DEV и PROD страницы
- Скриншот: `terraform destroy` — `Resources: 4 destroyed`

---

## Подсказка

Один модуль вызвали дважды с разными параметрами — это и есть суть переиспользования кода.
В реальных проектах так разворачивают десятки окружений (dev, staging, prod, qa)
из одного и того же модуля.
