# Транзакции, MVCC, ACID в MySQL 
создал три процедуры
```
CREATE PROCEDURE otusdb.create_user(IN p_user_login text)
BEGIN
  DECLARE v_login text;
  SELECT user_login INTO v_login FROM users WHERE user_login = p_user_login;
  IF v_login IS NULL
    THEN 
      -- Начало транзакции
      START TRANSACTION;
        INSERT INTO users (user_login,user_dg,user_st) values(p_user_login,now(),'ACTIVE');
      COMMIT;
  END IF;
END;

CREATE PROCEDURE otusdb.create_user_post(
  IN p_user_login text,
  IN p_post_text text,
  IN p_source_id int)
BEGIN
  DECLARE v_user_id int;
  DECLARE v_post_id int;
    
    SELECT user_id INTO v_user_id FROM users WHERE user_login = p_user_login;
    IF v_user_id IS NULL THEN
      START TRANSACTION;
        INSERT INTO users(user_login,user_dg,user_st)
          values(p_user_login,now(),'active');
      COMMIT;
      SELECT user_id INTO v_user_id FROM users WHERE user_login = p_user_login;
    END IF;

    SELECT post_id INTO v_post_id FROM post WHERE post_text = p_post_text;
    IF v_post_id IS NULL then
      START TRANSACTION;
        INSERT INTO post(post_text, post_desc, post_len,post_dt)
          values(p_post_text,'test',length(p_post_text),now());
      COMMIT;
      SELECT post_id INTO v_post_id FROM post WHERE post_text = p_post_text;
    END IF;

END;

CREATE PROCEDURE otusdb.create_user_inpost(
  IN p_user_login text,
  IN p_post_text text,
  IN p_source_id int)
BEGIN
  DECLARE v_user_id int;
  DECLARE v_post_id int;
  DECLARE v_inpost_id int;  
    SELECT user_id INTO v_user_id FROM users WHERE user_login = p_user_login;
    IF v_user_id IS NULL THEN
      START TRANSACTION;
        INSERT INTO users(user_login,user_dg,user_st)
          values(p_user_login,now(),'active');
      COMMIT;
      SELECT user_id INTO v_user_id FROM users WHERE user_login = p_user_login;
    END IF;

    SELECT post_id INTO v_post_id FROM post WHERE post_text = p_post_text;
    IF v_post_id IS NULL then
      START TRANSACTION;
        INSERT INTO post(post_text, post_desc, post_len,post_dt)
          values(p_post_text,'test',length(p_post_text),now());
      COMMIT;
      SELECT post_id INTO v_post_id FROM post WHERE post_text = p_post_text;
    END IF;
    
    SELECT inpost_id INTO v_inpost_id FROM inpost WHERE post_id = v_post_id AND user_id = v_user_id AND source_id = p_source_id;
    IF v_inpost_id IS NULL THEN 
      START TRANSACTION;
        INSERT INTO inpost(user_id, post_id,inpost_dt,source_id)
          values(v_user_id,v_post_id,now(),p_source_id);
      COMMIT;
    END IF;
END;
```
> Проверил
```
CALL create_user_post('ABRAMUS','test texta1',1);
CALL create_user_inpost('ABRAMUS','test texta1',1);
CALL create_user_post('AFANAS','test texta2',1);
CALL create_user_inpost('AFANAS','test texta3',1);
CALL create_user_inpost('ABRAMUS','test texta2',2);
```
![tr1](https://github.com/user-attachments/assets/c281f6f8-8424-42d5-ad40-8c6dcabc6783)
![tr2](https://github.com/user-attachments/assets/03fba79f-3a31-44cb-96eb-cf00f442dca9)
![tr3](https://github.com/user-attachments/assets/a9b9f428-fddd-4b09-8a8d-620d4e274f12)
![tr4](https://github.com/user-attachments/assets/9ca8586e-0976-4268-9026-8e19e1bdc487)
![tr5](https://github.com/user-attachments/assets/b30c1461-24ae-4674-b059-9b208288d527)
![tr6](https://github.com/user-attachments/assets/800f3061-0d54-4238-9d25-7aee5e48b2fc)
![tr7](https://github.com/user-attachments/assets/43c5d85c-4b92-4e4c-810d-4937337d3422)



