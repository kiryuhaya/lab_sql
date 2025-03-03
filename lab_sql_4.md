Лабораторная работа по УД номер 4

Подготовка: 

1. Используя код из лабораторной работы номер 3

    a) Создал нужные таблицы
   ![2025-03-03_20-50-29](https://github.com/user-attachments/assets/40e581f4-56e2-411b-9ded-b017a43b58cd)

    b) Вставил необходимые данные
   ![2025-03-03_20-50-39](https://github.com/user-attachments/assets/56dfa0ab-9d41-4b1e-9929-645d82194e97)


Ход работы:

 Задание 1:

Реализовать хранимую процедуру, возвращающую текстовую строку, содержащую информацию о технике (идентификатор, тип, дата, место и стоимость последней работы). Обработать ситуацию, когда техника не использовалась.
```
CREATE OR REPLACE FUNCTION get_tech_info(tech_id INT)
RETURNS TEXT AS $$
DECLARE
    tech_type VARCHAR(255);
    last_date VARCHAR(255);
    last_work_place VARCHAR(255);
    last_cost INT;
BEGIN
    IF tech_id IS NULL THEN
        RAISE NOTICE 'Ошибка: Идентификатор техники не может быть NULL';
        RETURN 'Ошибка: Передан NULL';
    END IF;

    SELECT ТИП INTO tech_type FROM Техника WHERE ИДЕНТИФИКАТОР = tech_id;
    IF NOT FOUND THEN
        RETURN FORMAT('Техника с ID %s не найдена', tech_id);
    END IF;

    SELECT z.ДАТА, m.НАЗВАНИЕ_ОРГ, z.ОПЛАТА_РУБ
    INTO last_date, last_work_place, last_cost
    FROM Заказ z
    JOIN Место_работы m ON z.МЕСТО_РАБОТЫ = m.ИДЕНТИФИКАТОР
    WHERE z.ТЕХНИКА = tech_id
    ORDER BY z.НОМЕР DESC
    LIMIT 1;

    IF last_date IS NULL THEN
        RETURN FORMAT('Техника %s (%s) не использовалась', tech_id, tech_type);
    ELSE
        RETURN FORMAT(
            'ID: %s, Тип: %s, Дата: %s, Место: %s, Стоимость: %s руб.',
            tech_id, tech_type, last_date, last_work_place, last_cost
        );
    END IF;
EXCEPTION
    WHEN others THEN
        RETURN 'Ошибка: ' || SQLERRM;
END;
$$ LANGUAGE plpgsql;
```

Примеры использования:

![2025-03-03_21-14-47](https://github.com/user-attachments/assets/60cad667-d55f-4365-9521-b64d905d12da)

![2025-03-03_21-15-04](https://github.com/user-attachments/assets/910c09d1-abb6-4ce7-8ad3-6c9502215076)

![2025-03-03_21-36-25](https://github.com/user-attachments/assets/b5fdd241-caee-478a-a8b7-511683330028)


 Задание 2:

Добавить таблицу, содержащую списки техники по каждому автопредприятию. При вводе заказа проверять наличие техники.

```
-- Таблица связи
CREATE TABLE Автопредприятие_Техника (
    АВТОПРЕДПРИЯТИЕ INT REFERENCES Автопредприятие(ИДЕНТИФИКАТОР),
    ТЕХНИКА INT REFERENCES Техника(ИДЕНТИФИКАТОР),
    PRIMARY KEY (АВТОПРЕДПРИЯТИЕ, ТЕХНИКА)
);
```

Вставил в нее значения, опираясь на уже известные данные

```
INSERT INTO Автопредприятие_Техника VALUES
(003, 007), 
(005, 007),
(002, 007),
(004, 002),
(006, 004),
(004, 004),
(001, 004),
(004, 003),
(005, 002),
(001, 006),
(003, 005),
(002, 001),
(005, 001),
(004, 001),
(006, 006),
(001, 001);
```

```
CREATE OR REPLACE FUNCTION check_tech_available()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.АВТОПРЕДПРИЯТИЕ IS NULL OR NEW.ТЕХНИКА IS NULL THEN
        RAISE EXCEPTION 'Не указаны обязательные поля';
    END IF;

    IF NOT EXISTS (
        SELECT 1 FROM Автопредприятие_Техника
        WHERE АВТОПРЕДПРИЯТИЕ = NEW.АВТОПРЕДПРИЯТИЕ
        AND ТЕХНИКА = NEW.ТЕХНИКА
    ) THEN
        RAISE EXCEPTION 'Техника % не доступна в автопредприятии %', NEW.ТЕХНИКА, NEW.АВТОПРЕДПРИЯТИЕ;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_tech_before_insert
BEFORE INSERT ON Заказ
FOR EACH ROW EXECUTE FUNCTION check_tech_available();
```
Пример использования: 

```
INSERT INTO Заказ (НОМЕР, ДАТА, МЕСТО_РАБОТЫ, АВТОПРЕДПРИЯТИЕ, ТЕХНИКА, КОЛ_ВО, ОПЛАТА_РУБ)
VALUES (053, 'Суббота', 002, 003, 007, 1, 100000);
```

![image](https://github.com/user-attachments/assets/621b6929-ba66-4fdf-b995-dc7b00d28e51)


 Задание 3:

Реализовать триггер такой, что при вводе строки в таблице заказов, если сумма не указана, то она вычисляется 

```
CREATE OR REPLACE FUNCTION calculate_payment()
RETURNS TRIGGER AS $$
DECLARE
    base_cost INT;
    discount INT;
BEGIN
    IF NEW.ОПЛАТА_РУБ = 0 THEN
        -- Проверка наличия данных
        SELECT СТОИМОСТЬ_ЗАКАЗА_РУБ INTO base_cost 
        FROM Техника 
        WHERE ИДЕНТИФИКАТОР = NEW.ТЕХНИКА;

        SELECT ЛЬГОТА INTO discount 
        FROM Место_работы 
        WHERE ИДЕНТИФИКАТОР = NEW.МЕСТО_РАБОТЫ;

        IF base_cost IS NULL THEN
            RAISE EXCEPTION 'Не найдена стоимость для техники %', NEW.ТЕХНИКА;
        END IF;

        NEW.ОПЛАТА_РУБ := (base_cost * NEW.КОЛ_ВО) * (1 - discount::NUMERIC / 100);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_calculate_payment
BEFORE INSERT OR UPDATE ON Заказ
FOR EACH ROW EXECUTE FUNCTION calculate_payment();
```
Пример использования: 

```
INSERT INTO Заказ (НОМЕР, ДАТА, МЕСТО_РАБОТЫ, АВТОПРЕДПРИЯТИЕ, ТЕХНИКА, КОЛ_ВО, ОПЛАТА_РУБ)
VALUES (054, 'Понедельник', 004, 003, 007, 2, 0);
```

![image](https://github.com/user-attachments/assets/3a44987d-b49c-4b9d-b512-f2d4e84f9efd)

```
INSERT INTO Заказ (НОМЕР, ДАТА, МЕСТО_РАБОТЫ, АВТОПРЕДПРИЯТИЕ, ТЕХНИКА, КОЛ_ВО, ОПЛАТА_РУБ)
VALUES (00022, 'Среда', 003, 002, 999, 1, 0);
```

![image](https://github.com/user-attachments/assets/f50e326b-951a-4c79-8dcf-899f6b7107e2)


 Задание 4:

Создать представление (view), содержащее поля: тип техники, дата, льгота, место работы, количество, стоимость заказа. Обеспечить возможность изменения предоставленной льготы. При этом должна быть пересчитана стоимость.

```
CREATE VIEW vw_заказ_льгота AS
SELECT 
    z.НОМЕР,
    t.ТИП AS тип_техники,
    z.ДАТА,
    m.ЛЬГОТА,
    m.НАЗВАНИЕ_ОРГ AS место_работы,
    z.КОЛ_ВО,
    z.ОПЛАТА_РУБ
FROM Заказ z
JOIN Техника t ON z.ТЕХНИКА = t.ИДЕНТИФИКАТОР
JOIN Место_работы m ON z.МЕСТО_РАБОТЫ = m.ИДЕНТИФИКАТОР;
```

```
CREATE OR REPLACE FUNCTION update_discount_view()
RETURNS TRIGGER AS $$
DECLARE
    base_cost INT;
    new_payment INT;
    order_tech INT;
    order_count INT;
    order_place INT;
BEGIN
    -- Получаем данные из соответствующей записи в Заказ
    SELECT ТЕХНИКА, КОЛ_ВО, МЕСТО_РАБОТЫ 
      INTO order_tech, order_count, order_place
    FROM Заказ
    WHERE НОМЕР = OLD.НОМЕР;

    -- Обновляем льготу в таблице Место_работы для данного места
    UPDATE Место_работы
    SET ЛЬГОТА = NEW.ЛЬГОТА
    WHERE ИДЕНТИФИКАТОР = order_place;

    -- Получаем базовую стоимость заказа для техники
    SELECT СТОИМОСТЬ_ЗАКАЗА_РУБ INTO base_cost
    FROM Техника 
    WHERE ИДЕНТИФИКАТОР = order_tech;

    -- Пересчитываем сумму с учётом новой льготы (процентное снижение)
    new_payment := (order_count * base_cost) * (100 - NEW.ЛЬГОТА) / 100;

    -- Обновляем сумму в таблице Заказ
    UPDATE Заказ
    SET ОПЛАТА_РУБ = new_payment
    WHERE НОМЕР = OLD.НОМЕР;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

```
CREATE TRIGGER trg_update_discount_view
INSTEAD OF UPDATE ON vw_заказ_льгота
FOR EACH ROW
EXECUTE FUNCTION update_discount_view();
```

Пример использования:

```
UPDATE vw_заказ_льгота
SET ЛЬГОТА = 15
WHERE НОМЕР = 012;
```
![image](https://github.com/user-attachments/assets/2764717a-e710-4fe1-9872-c614cc604c02)





   


    

