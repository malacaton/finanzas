# Finanzas personales
**App para gestión de finanzas (créditos y pagos) personales**

## DESCRIPCIÓN GENERAL ##
La clave también se almacenará como un hash, ya que no podrá descifrarse. Cada vez que el usuario se valide, se generará un hash con la clave insertada, y se contrastará con el hash almacenado. Puede utilizar un cifrado en un solo sentido, tipo un SHA-256 (no hace falta irse a 512) o bcrypt.

He valorado también cifrar los datos económicos con otro sistema que sí permita descifrado (marco los campos a cifrar con 'HIDE'). Lo mejor creo que podría ser utilizar el id del usuario para cifrar o descifrar sus datos. Podría haber sido un guid, por darle más complejidad. No obsante, como esto es una práctica de habilidades, he preferido dejar el id de usuario como un autonumérico, y no cifrar estos datos. Total, un administrador mañoso también podría descifrarlo, y en esta prueba no nos aportará mucho más.

## SEGURIDAD ##
Se autenticará mediante JWT. También tendrá un sencillo sistema de Roles, que en principio solo se usará para saber si el usuario es de tipo **Administrador** o **Usuario**. En caso de que el usuario validado sea de tipo **Administrador**, debería tener acceso a la gestión de usuarios, pero no a las pantallas de gestión de datos financieros. El tipo **Usuario**, por contra, no tendrá acceso a la gestión de usuarios, pero sí a las pantallas de gestión financiera, mostrando solamente los datos que le pertenecen.

## BD ##
La base de datos será MySQL, por no incurrir en gastos de licencias ya que es un proyecto personal. La BD se llamará **finanzas**, y estará codificada con **utf8_general_ci**.
La contraseña para hashear los passwords

## TABLAS ##
- users
  - id (guid)
  - name
  - username
  - password (hash creado con SHA, con SHA('contraseña') en MySQL. El SHA ocupa 40 caracteres, y no se puede decompilar)
  - role (Administrador o Usuario)
  - *... y lo que sea necesario. Lo estudiaremos*

- banks
  - id (pk)
  - user_id (fk)
  - banco		
		
- credits
  - id (pk)
  - user_id (fk)
  - bank_id (fk)
  - concept
  - capital (¿HIDE?)
  - on_cirbe (bit, default 0)
  - amount (¿HIDE?)
  - rate (nvarchar)
  - payment_day
  - last_payment_date
  - totatl_amortization_rate
  - partial_amortization_rate
  - note
  - amortization_table_created (bit, default false)
	
  *Cuando se cree un crédito, el valor 'amortization_table_created' será 0 (0=false en SQL). La pantalla debería permitirte ir a crear un cuadro de amortización en pantalla, que podrá modificar este dato.*
	
  *Esta pantalla debería permitir poner datos del crédito, en principio el día del primer pago, capital inicial e interés a aplicar, y al dar llamar a la acción creará registro en una subtabla. Estos registros serán editables, en principio creo que solo el importe del recibo, por el usuario, por si es necesario cuadrar algún dato (redondeos en céntimos o algo así). Al aceptar estos datos, se almacenarán en la tabla 'amortization_tables', y se cambiará el valor de 'cuadro_amortizacion_creado' a 1.*
	
- amortization_tables
  - id (pk)
  - user_id (fk)
  - credit_id (fk)
  - payment_date
  - real_payment_date (!= nulo indica "pagado")
  - capital (¿HIDE?)
  - rate (¿HIDE?)
  - amount (¿HIDE?) (se calculado inicialmente con cap. + int. y luego se podrá ajustar si falla algún céntimo)

- recurrent payments
  - id (pk)
  - user_id (fk)
  - bank_id (fk)
  - concept
  - amount (¿HIDE?)
  - pending_capital (Para tarjetas de crédito. Capital pendiente en la fecha indicada abajo)
  - pending_capital_date (Para tarjetas de crédito. Fecha en la que se comprobó el capital pendiente)
  - payment_date
  - months_between_payments (int, default 1)
  - last_payment_date (nullable)
  - note	
	
## PANTALLAS ##
- Gestión de usuarios (Administradores): Tabla con todos los usuarios, y su tipo de usuario.

  *Debería permitir modifiar datos, añadir o eliminar usuarios y administradores. Nunca permitirá eliminar al usuario actual (que es un administrador), aunque sí modificar su nombre y su password.*

- Menú inicial (Usuarios): Mostrará la pantalla de Estado Actual por defecto

- Estado Actual (el nombre se podrá revisar) (Usuarios): Un encolumnado con:
  - Banco
  - Concepto
  - Meses pago (campo meses_entre_pagos)
  - Importe
  - Imp. Mes (cálculo importe/meses_entre_pagos)
  - Dia pago
  - Pendiente
  - Fecha fin
  - Notas
	
  *Esta ventana debería poder mostrar también un "Sumatorio por banco", que es una subtabla dónde se muestra una fila por banco, con el total de recibos por banco, y también el capital pendiente sumado de todos los créditos.*
	
- Creditos (Usuarios): Gestión de Créditos. Enlace a "Cuadro de Amortización"
	
- Cuadro de Amortización (Usuarios): Se mostrará en subventana desde Creditos o en un popup.

  *Podría ser de pantalla completa, pero no se podrá llamar de forma independiente, sino desde el crédito. En caso de ser una ventana, debería poder volver al crédito, pero me gusta más la idea de un popup.*
	
- Pagos Recurrentes (Usuarios): Gestión de Pagos recurrentes

- Otras pantallas (Usuarios):

  *Estaría bien poder guardar simulaciones para amortizaciones anticipadas de créditos. Por ejemplo, crear una para incluir ciertos créditos seleccionados de la lista, incluyendo aquellos pagos recurrentes que tengan un capital_pendiente (normalmente serán tarjetas de crédito). Esto mostrará una tabla con los datos de solo esos créditos y en el sumatorio se verá el total/mes (para saber lo que dejarás de pagar al mes), y el importe total (para saber el montante que hará falta para cancelarlo, teniendo en cuenta el % sobre el capital pendiente que se aplicacará por amortización total).*

  *Por ejemplo, yo tengo guardada una hoja dónde me aparecen los créditos que yo he seleccionado, pero solo de entre los que salen en CIRBE, que son los primeros que me quiero quitar. Tengo otra dónde me salían las cancelaciones de las tarjetas de crédito. Luego puedes hacer otra para ir jugando, como decir "¿cuanto me costaría cancelar este crédito, aquella tarjeta y aquella otra?". Eso lo he usado yo mucho para ir guardando, o intentando conseguir otros créditos más favorables, que me permitiesen cancelar mis tarjetas. A día de hoy, ya las he conseguido cancelar.*
	
  *Podríamos estudiar, pero eso más adelante, hacer otra para mostrar una amortización parcial de uno o varios créditos, para que calcule el importe a pagar teniendo en cuenta el % de penalización por amortización anticipada.*

