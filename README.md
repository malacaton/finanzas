# Finanzas personales
**App para gestión de finanzas (créditos y pagos) personales**

## DESCRIPCIÓN GENERAL ##
La base de datos será MySQL, por no incurrir en gastos de licencias ya que es un proyecto personal.

La clave también se almacenará como un hash, ya que no podrá descifrarse. Cada vez que el usuario se valide, se generará un hash con la clave insertada, y se contrastará con el hash almacenado. Puede utilizar un cifrado en un solo sentido, tipo un SHA-256 (no hace falta irse a 512) o bcrypt.

Cifraremos también los datos económicos con otro sistema que sí permita descifrado (marco los campos a cifrar con (HIDE). Lo mejor creo que podría ser utilizar el guid del usuario para cifrar o descifrar sus datos. No obsante, como esto es una práctica de habilidades, Esto hace que el administrador mañoso también pudiese descibrarlo, pero ya es una muestra de capacidad, y de que pensamos en esas cosas. Siendo así, si algún día mostramos esto a alguien podríamos decir que eso no es privado al 100%, pero que en un planteamiento profesional se estudiaría mejor. O estudiarlo mejor, de entrada, y hacerlo ya - XDD.

## SEGURIDAD ##
Se autenticará mediante JWT. También tendrá un sencillo sistema de Roles, que en principio solo se usará para saber si el usuario es de tipo **Administrador** o **Usuario**. En caso de que el usuario validado sea de tipo **Administrador**, debería tener acceso a la gestión de usuarios, pero no a las pantallas de gestión de datos financieros. El tipo **Usuario**, por contra, no tendrá acceso a la gestión de usuarios, pero sí a las pantallas de gestión financiera, mostrando solamente los datos que le pertenecen.

## TABLAS ##
- usuarios
  - id (guid)
  - nombre
  - user
  - pass (hash)
  - rol (Administrador o Usuario)
  - *... y lo que sea necesario. Lo estudiaremos*

- bancos
  - id (pk)
  - user_id (fk)
  - banco		
		
- creditos
  - id (pk)
  - user_id (fk)
  - banco_id (fk)
  - concepto
  - capital (¿HIDE?)
  - en_cirbe (bit, default 0)
  - importe_recibo (¿HIDE?)
  - tipo_interes (nvarchar)
  - dia_pago
  - fecha_ultimo_pago
  - porcentaje_amortizacion_total
  - porcentaje_amortizacion_parcial
  - notas
  - cuadro_amortizacion_creado (bit, default false)
	
  *Cuando se cree un crédito, el valor 'cuadro_amortizacion_creado' será 0 (0=false en SQL). La pantalla debería permitirte ir a crear un cuadro de amortización en pantalla, que podrá modificar este dato.*
	
  *Esta pantalla debería permitir poner datos del crédito, en principio el día del primer pago, capital inicial e interés a aplicar, y al dar llamar a la acción creará registro en una subtabla. Estos registros serán editables, en principio creo que solo el importe del recibo, por el usuario, por si es necesario cuadrar algún dato (redondeos en céntimos o algo así). Al aceptar estos datos, se almacenarán en la tabla 'amortizaciones_creditos', y se cambiará el valor de 'cuadro_amortizacion_creado' a 1.*
	
- amortizaciones_creditos
  - id (pk)
  - user_id (fk)
  - credito_id (fk)
  - fecha_pago
  - fecha_pago_real (!= nulo indica "pagado")
  - capital (¿HIDE?)
  - interes (¿HIDE?)
  - importe (¿HIDE?) (se calculado inicialmente con cap. + int. y luego se podrá ajustar si falla algún céntimo)

- pagos_recurrentes
  - id (pk)
  - user_id (fk)
  - banco_id (fk)
  - concepto
  - importe_recibo (¿HIDE?)
  - capital_pendiente (Para tarjetas de crédito. Capital pendiente en la fecha indicada abajo)
  - fecha_capital_pendiente (Para tarjetas de crédito. Fecha en la que se comprobó el capital pendiente)
  - dia_pago
  - meses_entre_pagos (int, default 1)
  - fecha_ultimo_pago (nullable)
  - notas	
	
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

