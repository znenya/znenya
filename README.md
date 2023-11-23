### Hi there 👋

Привет! Меня зовут Евгений, я начинающий аналитик данных. В этом репозитории вы можете найти некоторые из моих проектов, выполненных во время обучения и практики.

### Навыки и технологии
 
Инструменты анализа данных: SQL, Excel

Языки программирования и библиотеки: Python

Системы управления базами данных: MySQL, PostgreSQL

Средства визуализации данных: PowerBi

### Проекты

#### Проект 1: Калькулятор юнит-экономики онлайн-кинотеатра

Курсовой проект, в котором я анализировал и строил калькулятор юнит-экономики онлайн-кинотеатра. Подробности Вы можете посмотреть в файле "Курсовая онлайн-кинотеатр".

#### Проект 2: Моделирование изменения балансов уроков онлайн-школы

В этой работе я писал SQL-запрос для определения изменения баланса уроков студентов онлайн-школы.

<details>
<summary>SQL-запрос</summary>

```
with first_payments as (
       select user_id,
             min(transaction_datetime)::date as first_payment_date
    from skyeng_db.payments
       where status_name = 'success'
     group by user_id
),

all_dates as (
        select distinct class_start_datetime::date as dt 
from skyeng_db.classes
where class_start_datetime between '2016-01-01 00:00' and '2016-12-31 23:59'
),


    all_dates_by_user as (
       select user_id,
               dt
    from all_dates ad
      join first_payments fp
      on ad.dt::date >= fp.first_payment_date::date
),

 payments_by_dates as (
      select user_id,
      transaction_datetime::date as payment_date,
      sum(classes) as transaction_balance_change
from skyeng_db.payments
where status_name = 'success'
group by user_id,
       payment_date
),

payments_by_dates_cumsum as (
     select adbu.user_id,
            dt,
            transaction_balance_change,
            sum(coalesce (transaction_balance_change, 0)) over (partition by adbu.user_id order by dt) as transaction_balance_change_cs 
        from all_dates_by_user as adbu
             left join payments_by_dates as pbd
               on adbu.user_id = pbd.user_id
               and adbu.dt = pbd.payment_date
            order by dt
),

   classes_by_dates as (
       select user_id,
             class_start_datetime::date as csdt,
             count(*) * (-1) as classes
    from skyeng_db.classes
    where class_type <> 'trial' and class_status in ('success', 'failed_by_student')
    group by user_id, csdt
),

    classes_by_dates_dates_cumsum as (
       select adbu.user_id,
               dt,
               classes,
               sum(coalesce (classes,0)) over (partition by adbu.user_id order by dt) as classes_cs
        from all_dates_by_user adbu
        left join classes_by_dates cbd
          on adbu.user_id = cbd.user_id
          and adbu.dt = cbd.csdt
),

      balances as (
    select pbdc.user_id,
            pbdc.dt,
            transaction_balance_change,
            transaction_balance_change_cs,
            classes,
            classes_cs,
            classes_cs + transaction_balance_change_cs as balance
        from payments_by_dates_cumsum pbdc
        left join classes_by_dates_dates_cumsum cbddc
        on pbdc.user_id = cbddc.user_id
        and pbdc.dt = cbddc.dt
)

-- select *
-- from balances
-- limit 1000
-- Есть количество уроков отрицательное.


    
    select dt,
        sum (transaction_balance_change) as tbc_sum,
        sum(transaction_balance_change_cs) as tbcc_sum,
        sum (classes) as classes_by_dates_dates_cum,
        sum (classes_cs) as classes_cs_sum,
        sum (balance) as balance_sum
    from balances
    group by dt
    order by dt
    
--  Задание 2. Выражена сезонность, большие пики весной, летом идет на спад.

    
    
     
         
    




```

</details>

