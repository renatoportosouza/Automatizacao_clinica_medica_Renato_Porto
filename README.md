# Projeto de Banco de Dados - Clínica Médica - Programador FullStack (SENAI)

## Descrição
Este projeto faz parte do curso de Programador FullStack do SENAI e tem como objetivo implementar automatizações em um banco de dados de uma clínica médica utilizando **Triggers** e **Stored Procedures** no **SQL Server Management Studio (SSMS)**.

As automatizações incluem:
- Cálculo automático do valor a ser pago nos pedidos de exames de acordo com o plano de saúde.
- Consulta de agenda de médicos.
- Consulta de exames solicitados.
- Consulta do histórico e resumo de pagamentos.

## Requisitos
- SQL Server Management Studio (SSMS).
- Banco de dados `clinica_medica` criado e populado conforme o script fornecido (`tb_clinica_medica.sql`).
- Permissões para criar triggers e stored procedures.

## Passo a Passo
### 1. Configurar o Banco de Dados
1. Abra o **SSMS** e clique em **Nova Consulta**.
2. Selecione o banco de dados digitando:
   ```sql
   USE clinica_medica;
   ```
3. Execute o comando:
   ```sql
   SELECT * FROM pedido_exame;
   ```
   Isso mostrará os dados da tabela `pedido_exame`.

### 2. Criar a Trigger `Atualiza_Pedido_Exame`
#### 2.1 Apagar os dados antigos
1. Abra o script `tb_clinica_medica.sql` e remova o comentário da linha:
   ```sql
   DELETE FROM pedido_exame;
   ```
2. Execute o comando para apagar os dados.
3. Verifique se os dados foram apagados executando:
   ```sql
   SELECT * FROM pedido_exame;
   ```

#### 2.2 Reiniciar a contagem dos IDs
1. Execute:
   ```sql
   DBCC CHECKIDENT('pedido_exame', RESEED, 2199);
   ```

#### 2.3 Criar a Trigger
1. Execute o seguinte código SQL para criar a trigger:
   ```sql
   CREATE TRIGGER Atualiza_Pedido_Exame
   ON pedido_exame
   AFTER INSERT
   AS
   BEGIN
       SET NOCOUNT ON;
       DECLARE @num_ped INT;
       SELECT @num_ped = numero_pedido FROM inserted;
       DECLARE @num_cons INT;
       SELECT @num_cons = fk_consulta_numero_consulta FROM inserted;
       DECLARE @cod_ex INT;
       SELECT @cod_ex = fk_exame_codigo FROM inserted;
       DECLARE @prc MONEY;
       SELECT @prc = preco FROM exame WHERE codigo = @cod_ex;
       DECLARE @cpf_pac VARCHAR(20);
       SELECT @cpf_pac = fk_paciente_cpf FROM consulta WHERE numero_consulta = @num_cons;
       DECLARE @tp_plan VARCHAR(20);
       SELECT @tp_plan = tipo_plano FROM paciente WHERE cpf = @cpf_pac;
       
       IF @tp_plan = 'Especial'
           UPDATE pedido_exame SET valor_pagar = 0 WHERE numero_pedido = @num_ped;
       ELSE IF @tp_plan = 'Padrão'
           UPDATE pedido_exame SET valor_pagar = @prc * 0.7 WHERE numero_pedido = @num_ped;
       ELSE IF @tp_plan = 'Básico'
           UPDATE pedido_exame SET valor_pagar = @prc * 0.9 WHERE numero_pedido = @num_ped;
       
       PRINT 'Trigger (Atualiza Pedido de Exame) Encerrada';
   END;
   ```
2. Selecione todo o script e clique em **Executar**.
3. Verifique se a trigger foi criada indo até a pasta **Gatilhos** dentro da tabela `pedido_exame` no SSMS.

### 3. Criar Stored Procedures
#### 3.1 `Agenda_Medicos`
1. Execute o seguinte comando para criar a stored procedure:
   ```sql
   CREATE PROCEDURE Agenda_Medicos
   AS
   BEGIN
       SELECT m.nome_medico, m.especialidade, m.crm, c.numero_consulta, 
              c.data_consulta, c.horario_consulta, p.nome_paciente, p.cpf, 
              p.nome_plano, p.tipo_plano 
       FROM medico AS m
       INNER JOIN consulta AS c ON m.crm = c.fk_medico_crm
       INNER JOIN paciente AS p ON c.fk_paciente_cpf = p.cpf
       ORDER BY m.nome_medico, c.data_consulta;
   END;
   ```
2. Execute a stored procedure:
   ```sql
   EXECUTE Agenda_Medicos;
   ```

#### 3.2 `Exames_Solicitados`
1. Execute o seguinte comando:
   ```sql
   CREATE PROCEDURE Exames_Solicitados
   AS
   BEGIN
       SELECT pe.numero_pedido, pe.data_exame, pe.valor_pagar, c.numero_consulta, p.nome_paciente
       FROM pedido_exame AS pe
       INNER JOIN consulta AS c ON pe.fk_consulta_numero_consulta = c.numero_consulta
       INNER JOIN paciente AS p ON c.fk_paciente_cpf = p.cpf;
   END;
   ```
2. Execute a stored procedure:
   ```sql
   EXECUTE Exames_Solicitados;
   ```

#### 3.3 `Historico_Pagamentos`
1. Execute o seguinte comando:
   ```sql
   CREATE PROCEDURE Historico_Pagamentos
   AS
   BEGIN
       SELECT p.nome_paciente, SUM(pe.valor_pagar) AS total_pago
       FROM paciente AS p
       INNER JOIN consulta AS c ON p.cpf = c.fk_paciente_cpf
       INNER JOIN pedido_exame AS pe ON c.numero_consulta = pe.fk_consulta_numero_consulta
       GROUP BY p.nome_paciente;
   END;
   ```
2. Execute a stored procedure:
   ```sql
   EXECUTE Historico_Pagamentos;
   ```

## Conclusão
Com este projeto, conseguimos aplicar conceitos avançados de **Triggers** e **Stored Procedures** para automatizar processos dentro do banco de dados de uma clínica médica. 


