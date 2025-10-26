## 1. Register in a cloud provider. Now you have a root account (1 point)
![](screenshots/1-register.png)

### 1.1. Create an IaM Identity center account (or analog for you cloud provider). Also make it full admin. (2 points)

В пошуку знаходимо IaM Identity Center та переходимо на головну сторінку - Dashboard.
![](screenshots/1.1-iam-identity-dashboard.png)

Тепер створимо декілька користувачів.
В процесі їх створення треба заповнити інформацію про цих користувачів,
та після створення активувати через посилання, що прийде на електронну адресу, як лист запрошення.

![](screenshots/1.1-iam-create-user.png)
![](screenshots/1.1-create-user2.png)

Після цієї процедури бачимо пусту сторінку користувача в AWS access portal.
![](screenshots/1.1-aws-access-portal.png)

Тепер перейдемо до Permission Sets, щоб або вибрати готові права або створити власні

Створимо AdminAccess Permission Set та прив'яжемо до нашого акаунту.
Для цього у вкладці PermissionSets вибираємо Predefined Permission Set -> AdminAccess
![](screenshots/1.1-admin-permision-set.png)
![](screenshots/1.1-admin-permision-set2.png)

Далі у вкладці Aws accounts вибираємо Assign users or groups та надаємо користувачу права адміна
![](screenshots/1.1-account-admin-access.png)
---

## 2. Create Organization 
Оскільки при створенні сервісу IaM Identity Center організація створюється автоматично, 
то створимо новий акаунт з доступом до організації

![img.png](screenshots/2-create-organization.png)

---

## 3. Create one more user. Create an IaM policy that allows ONLY to view resources (no write access). (1 point)

Створимо ReadOnly Access Policy
![](screenshots/1.1-readonly-policy.png)
![](screenshots/1.1-readonly-policy2.png)

Оскільки користувача ми створили вище, то просто дамо йому ці політики доступу
![](screenshots/3-provision-readonly-policy.png)
![](screenshots/3-provision-readonly-policy2.png)


---

# 4. Create a Role that has this policy attached and can be assumed by user from p3. (1 point) 

Створимо в IAM ReadOnlyUserRole
![](screenshots/4-readonly-user-role.png)

Далі коли маємо певну роль, створимо новий Permission Set (ExistingRolePermisionSet) 
в якому явно пропишемо роль, яку користувач може використовувати
![](screenshots/4-assume-readonly-role.png)

Але попередньо в IAM треба прописати, що ця роль має Trust Relationships з створеним Permission Set
![](screenshots/4-trust-relationships.png)

Перевіримо.

Заходимо в наш TestAccount, який має ExistingRolePermissionSet
![](screenshots/4-existing-permission-signin.png)

Тепер змінимо роль використовуючи панель справа вгорі
![](screenshots/4-switch-user-role.png)

Як бачимо, роль умпішно змінено
![](screenshots/4-switch-role-result.png)

---

# 5. Cloudformation

Опишемо тепер ролі та policy через Infrastucture as a Code, в моєму випадку - Cloudfomation
![](screenshots/5-cloudformation-new.png)

Як результат запуску WriteRolePolicy, отримуємо 2 успішно створені ресурси:

А саме - AssumeWriteAccessRolePolicy та WriteAccessUserRole
![](screenshots/5-cloudformation-job-result.png)
![](screenshots/5-cloudformation-resources.png)

---

# 6. VPC

Створимо тепер Virtual Private Cloud використовуючи AWS VPC

Першим кроком створимо саму VPC, це робиться досить легко, 
на цьому етапі лише важливо прописати CIDR, щоб виділити певну кількість ip-адрес.
Було обрано дефолтне значення 10.0.0.0/24, тобто ми виділили простір адрес з 10.0.0.0 - 10.0.0.255 
![](screenshots/6-create-vpc.png)


Наступним кроком створимо публічну на приватні підмережі - вкладка Subnets. 

Назвемо їх vpc-lab1-public. CIDR - 10.0.0.0/28 (10.0.0.0 - 10.0.0.15)
![](screenshots/6-vpc-public.png)

Та vpc-lab1-private. CIDR - 10.0.0.16/28 (10.0.0.16 - 10.0.0.31)
![](screenshots/6-vpc-private.png)

Тепер створимо Internet Gateway, щоб публічна підмережа мала доступ в інтернет
![](screenshots/6-igw.png)

Але для того, щоб він почав працювати треба налаштувати Route Tables. Зробимо це для обох підмереж

Приватна підмережа, буде працювати в межах створеної VPC
![](screenshots/6-vpc-private-rtb.png)

А публічна в межах VPC та матиме доступ в інтернет через створений Internet Gateway
![](screenshots/6-vpc-public-rtb.png)

---

# 7. IaC VPC

Використаємо для цього готовий інструмент від AWS для генерації IaC - назвивається AWS IaC Generator

Відскануємо ресурси
![](screenshots/7-iac-scan-resources-0.png)
![](screenshots/7-iac-scan-resources.png)

Після вибору всіх необхідних ресурсів, AWS автоматично згенерує файл інфрастуктури як коду
![img.png](screenshots/7-iac-select-resources.png)

Отримуємо результат створеного темплейту
![img.png](screenshots/7-iac-generate-result.png)

---

# 8. Calculate monthly budget