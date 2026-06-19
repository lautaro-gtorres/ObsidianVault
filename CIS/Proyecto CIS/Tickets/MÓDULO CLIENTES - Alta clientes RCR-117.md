te comento que hay un error con los datos de un cliente que creamos al momento de hacer el envío, el campo de país de residencia está en blanco y bloqueado.
![[Pasted image 20260512102046.png]]  

Sin embargo, si me voy a editar al cliente sí están los datos. Para mí que no está recuperando bien la información registrada.
![[Pasted image 20260512102319.png]]    

Resumen:
Se creó un cliente nuevo y se verifica que no recupera el país de residencia del cliente (en datos maestros clientes se verifica que el cliente sí cuenta con dicha información).

Datos Adicionales:
Es muy probable que sea un tema de ProviderId Se crean con un numero de provider y despues en la pantalla de envios se usa otro. Deberia haber una funcion intermedia que haga la conversion pero parece que esta fallando podrias revisar por ahi.