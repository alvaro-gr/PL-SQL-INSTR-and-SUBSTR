# Parsing-with-PL-SQL-INSTR-and-SUBSTR-

BEGIN

    req := utl_http.begin_request(string_url); 

    utl_http.set_header(req, 'User-Agent', 'Mozilla/4.0');

    resp := utl_http.get_response(req);
    DBMS_OUTPUT.PUT_LINE('HTTP response status code: ' || resp.status_code);
 

     IF resp.status_code = 200 THEN
        BEGIN
            LOOP
                utl_http.read_raw(resp, valor);  
            END LOOP;
            utl_http.end_response(resp);
        EXCEPTION
            WHEN utl_http.end_of_body THEN
                utl_http.end_response(resp);
            WHEN others THEN
             valor:=null;
             
        END; 
        
        DELETE FROM my_table;
        
        WHILE seguir LOOP
            pos_ini := INSTR(valor,'4D41545',1,contador);
            pos_fin := (INSTR(valor,'740A',1,contador))+3;
            tam := (pos_fin-pos_ini)+1;
            
            IF pos_ini <> 0 THEN
                  
                fichero := utl_raw.cast_to_varchar2(SUBSTR(valor, pos_ini, tam));
                pre_fecha := utl_raw.cast_to_varchar2(SUBSTR(valor, pos_ini-26, 26));
                dat1 := SUBSTR(pre_fecha, 1, 3);
                dat2 := SUBSTR(pre_fecha, 5, 2);
                dat3 := dat1 || '-' || dat2;
                dat :=  to_char(to_date(dat3,'Mon-DD'),'YYYY-MM-DD');
                pre_time := SUBSTR(pre_fecha, 8, 5);
                tim := to_char(to_date(pre_time,'HH24:MI'),'HH24:MI:SS');
                fecha:= dat || ' ' || tim;
                fecha_correcta := to_date(fecha, 'YYYY/MM/DD HH:MI:SS');
                
                INSERT INTO my_table (nombre_fichero, fecha) VALUES (fichero, fecha_correcta);
                          
                contador := contador+1;
                
            ELSE
                seguir := FALSE;
            END IF;
         END LOOP;   
         
         COMMIT;   
                             
    ELSE
        DBMS_OUTPUT.PUT_LINE('ERROR');
        utl_http.end_response(resp);
    END IF;

END;
