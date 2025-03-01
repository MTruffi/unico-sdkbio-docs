---
sidebar_label: 'Reconhecimento facial'
sidebar_position: 4
---

# Reconhecimento facial

## Sobre este guia

Este guia foi elaborado para ajudá-lo a integrar nosso SDK Android de forma rápida e fácil. 
Buscamos trazer conceitos básicos, exemplos de implementação dos SDKs e também de como interagir com as APIs REST de nosso motor biométrico.

:::info Vale lembrar
Vale lembrar que este guia foca no processo de captura de imagens. Caso esteja buscando informações sobre as APIs REST do **Unico Check**, sugerimos que visite nosso [API Reference](api), nosso guia de APIs ou nossa página de página [Visão Geral](overview).
:::

Através deste guia, em poucos minutos você será capaz de:

- Implementar a abertura da câmera e captura de imagem;
- Entender como manipular os dados de retorno;
- Entender como utilizar o retorno de nosso SDK com nossas APIs;

## Antes de começar

Certifique-se que você seguiu nosso passo-a-passo para instalação e importação de nosso SDK através [deste guia](como-comecar). É importante também ter em conta as funcionalidades disponíveis neste SDK, como explicado na página de [Visão Geral](overview) deste SDK.

## Recursos disponíveis

Nosso SDK oferece um componente para captura de imagem contendo uma silhueta que ajuda o usuário a se posicionar de forma correta para a foto. A captura da imagem para o reconhecimento facial pode ser feita de duas formas, descritas ao longo desse guia. Sendo elas:

### Captura Manual

Neste tipo de experiência seu usuário é totalmente responsável por posicionar sua face dentro da área de captura. Após se posicionar corretamente, o usuário deve clicar em um botão para capturar a imagem. 

Neste tipo de captura, nosso SDK não efetua nenhum tipo de validação do que está sendo capturado e isso pode aumentar as chances de problemas ao enviar o `base64` obtido para as APIs de nosso motor de biometria.

import imgCapturaManual from '/static/img/guias/img_cameranormal.png';

<img src={imgCapturaManual} alt="Captura Manual" className="imgCenter" />


### Captura Automática

Neste tipo de experiência, identificamos a face do usuário automáticamente através de algorítimos de visão computacional e o auxiliamos para que se posicione de forma correta dentro da área de captura. Após se posicionar corretamente, capturamos a imagem de forma automática.

Por ajudar o usuário a enquadrar sua face na área de captura, esta opção pode diminuir problemas ao enviar o `base64` às APIs de nosso motor biométrico.

import imgCapturaAutomatica from '/static/img/guias/img_camerainteligente.png';

<img src={imgCapturaAutomatica} alt="Captura Manual" className="imgCenter" />

## Implementação

Ao seguir este passo-a-passo, em poucos minutos você terá todo o potencial de nosso SDK embarcado em sua aplicação Android.

import Steps from '@site/src/components/Steps';

<Steps headingDepth={3}>
<ol>
<li>

### Inicializar nosso SDK

Para iniciar, crie uma instância do builder (gerado através da interface `IAcessoBioBuilder`), fornecendo como parâmetro o *contexto* em questão e a implementação da classe [`AcessoBioListener`](#). 

A implementação dessa classe é bem simples e pode ser feita com poucas linhas de código. Tudo que precisa fazer é sobrescrever nossos métodos de callback com as lógicas de negócio de sua aplicação.

:::info Documentação Adicional
Você pode encontrar mais detalhes sobre a classe [AcessoBioListener](#) e sua implementação em nosso API Reference.
:::

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
  <TabItem value="java" label="Java" default>

```java
public class MainActivity extends AppCompatActivity implements AcessoBioListener {

    private AcessoBioListener callback = new AcessoBioListener() {
        @Override
        public void onErrorAcessoBio(ErrorBio errorBio) { }

        @Override
        public void onUserClosedCameraManually() { }

        @Override
        public void onSystemClosedCameraTimeoutSession() { }

        @Override
        public void onSystemChangedTypeCameraTimeoutFaceInference() { }
    };

    private IAcessoBioBuilder acessoBioBuilder = new AcessoBio(this, callback);
}
```

  </TabItem>

  <TabItem value="kotlin" label="Kotlin">

```kotlin
internal class MainActivity : AppCompatActivity() {

    private val callback = object : AcessoBioListener {
        override fun onErrorAcessoBio(errorBio: ErrorBio?) { }
    
        override fun onUserClosedCameraManually() { }
    
        override fun onSystemClosedCameraTimeoutSession() { }
    
        override fun onSystemChangedTypeCameraTimeoutFaceInference() { }
    }

    private val acessoBioBuilder: IAcessoBioBuilder = AcessoBio(this, callback)
}
```

  </TabItem>
</Tabs>


</li>

<li>

### Implementar funções de callback

Como mencionado no passo anterior, o trabalho de implementação da classe [AcessoBioListener](#) é, em grande parte, a configuração dos métodos de callback. Cada método será chamado em uma situação específica de retorno de nosso SDK, como detalhado abaixo. 

Basta sobrescrever os métodos exemplificados no passo anterior com as lógicas de negócio de sua aplicação. Saiba um pouco mais sobre os métodos de retorno:

#### Método `onErrorAcessoBio(ErrorBio errorBio)`
Este método será invocado sempre quando qualquer erro de implementação ocorrer ao utilizar algum de nossos métodos, como por exemplo, ao informar um tipo de documento incorreto para a funcionalidade de captura de documentos.

Ao ser invocado, o método receberá um parâmetro do tipo `ErrorBio` que contem detalhes do erro. Saiba mais sobre o tipo `ErrorBio` no [API Reference](#) de nosso SDK.

#### Método `onUserClosedCameraManually()`
Este método será invocado sempre quando o usuário fechar a câmera de forma manual, como por exemplo, ao clicar no botão "Voltar".

#### Método `onSystemClosedCameraTimeoutSession()`
Este método será invocado assim que o tempo máximo de sessão for atingido (Sem capturar nenhuma imagem).

<!-- TODO Arrumar -->

:::info  Tempo máximo da sessão
O tempo máximo da sessão pode ser configurado em nosso **builder** através do método `setTimeoutSession`. Saiba mais sobre  método `setTimeoutSession()` no [API Reference](#) de nosso SDK.

:::

#### Método `onSystemChangedTypeCameraTimeoutFaceInference()`
Este método será invocado assim que o tempo máximo para detecção da face de um usuário for atingido (sem ter nada detectado). Neste caso, o modo de câmera é alterado automaticamente para o modo manual (sem o smart frame).

Para mais detalhes sobre os métodos de `callback`, consulte nossa a [API Reference](api/callback) de nosso SDK Android.

:::caution Atenção

Todos os métodos acima devem devem ser criados da forma indicada em seu projeto (mesmo que sem nenhuma lógica). Caso contrário, o projeto não compilará com sucesso.

:::


</li>


<li>

### Configurar modo da câmera

Em seguida, deve-se decidir e configurar qual modo de camerâ será utilizado em seu aplicativo. Como explicamos [acima](reconhecimento-facial#recursos-disponíveis) existem dois modos de captura disponíveis.  

Nosso SDK tem configurado e habilitado por padrão o *enquadramento inteligente* e a *captura automática*. Para utilizar a câmera em modo normal, desabilite ambas funcionalidades através dos métodos `setAutoCapture` e `setSmartFrame`. Os exemplos a seguir demonstram como você pode configurar cada um dos modos de câmera.

----

#### Modo inteligente (Captura automática - Smart Camera)

Por padrão, nosso SDK possui o enquadramento inteligente e a captura automática habilitados. Caso decida utilizar este modo de câmera, não será necessário alterar nenhuma configuração. 

Caso as configurações da câmera tenham sido alteradas previamente em seu App, é possível restaurá-las através dos métodos `setAutoCapture` e `setSmartFrame`:

<Tabs>
<TabItem value="java" label="Java" default>

```java

UnicoCheckCamera unicoCheckCamera = acessoBioBuilder
    .setAutoCapture(true)
    .setSmartFrame(true)
    .build();

```
</TabItem>

<TabItem value="kotlin" label="Kotlin">


```kotlin

val unicoCheckCamera: UnicoCheckCamera = acessoBioBuilder
    .setAutoCapture(true)
    .setSmartFrame(true)
    .build()

```

</TabItem>
</Tabs>


:::caution Atenção

Não é possível implementar o método `setAutoCapture(true)` com o método `setSmartFrame(false)`. Ou seja, não é possível manter a captura automática sem o Smart Frame, pois ele é quem realiza o enquadramento inteligênte.

:::

#### Modo manual 

Por padrão, nosso SDK possui o enquadramento inteligente e a captura automática habilitados. Neste caso, para utilizar o modo manual ambas configurações relacionadas a *Smart Camera* devem ser desligadas através dos métodos `setAutoCapture` e `setSmartFrame`:

<Tabs>
  <TabItem value="java" label="Java" default>

```java {2-3}
UnicoCheckCamera unicoCheckCamera = acessoBioBuilder
    .setAutoCapture(false)
    .setSmartFrame(false)
    .build();
```
  </TabItem>

  <TabItem value="kotlin" label="Kotlin">

```kotlin {2-3}
val unicoCheckCamera: UnicoCheckCamera = acessoBioBuilder
    .setAutoCapture(false)
    .setSmartFrame(false)
    .build()
```

  </TabItem>
</Tabs>

:::tip Dica - SmartFrame

Mesmo em modo manual é possível utilizar o Smart Frame. Neste caso, exibiremos a silhueta para identificar o enquadramento para então habilitar o botão. Para isto, basta configurar `setAutoCapture=false` e `setSmartFrame=true`

:::

</li>
<li>

### Efetuar abertura da câmera

O último passo é disparar a abertura da câmera. Vamos dividir este processo em algumas etapas:

#### Implementar listeners para eventos da câmera

O método de abertura da câmera precisa saber o que fazer ao conseguir capturar uma imagem ou ao ter algum erro no processo. Informaremos "o que fazer" ao método de abertura da câmera através da implantação de *listeners* que serão chamados em situações de sucesso ou erro.

Através da implementação dos *listeners*, você poderá especificar o que acontecerá em seu App em situações de erro (método `onErrorSelfie`) ou sucesso (método `onSuccessSelfie`) na captura de imagens.

##### Método `onSuccessSelfie`

Ao efetuar uma captura de imagem com sucesso, este método será invocado e retornará um objeto do tipo [`ResultCamera`](#) que será utilizado posteriormente na chamada de nossas APIs REST. Saiba mais sobre o tipo `ResultCamera` no [API Reference](#) de nosso SDK.

##### Método `onErrorSelfie`

Ao ocorrer algum erro na captura de imagem, este método será invocado e retornará um objeto do tipo [`ErrorBio`](#). Saiba mais sobre o tipo `ResultCamera` no [API Reference](#) de nosso SDK.

:::note Implementação dos listeners
A implementação destes métodos (*listeners*) deverá ser feita através de uma instância da classe [`iAcessoBioSelfie`](#). Saiba mais sobre a classe `iAcessoBioSelfie` no [API Reference](#) de nosso SDK.
:::

#### Preparar a câmera
Preparar a câmera para abertura utilizando o método `prepareSelfieCamera`. Este método retornará um objeto do tipo `UnicoCheckCameraOpener.Selfie`, que quando retornado será utilizado para a abertura da câmera;

#### Abrir a câmera
Abrir a câmera com o objeto do tipo `UnicoCheckCameraOpener.Selfie`, utilizando o método `open` fornecendo os *listeners* como parâmetro.

--- 

#### Juntando todos os passos

Ao juntar todos os passos, a configuração dos listeners e abertura ficará da seguinte forma:



 <Tabs>
  <TabItem value="java" label="Java" default>

```java
iAcessoBioSelfie cameraListener = new iAcessoBioSelfie() {
    @Override
    public void onSuccessSelfie(ResultCamera result) { }

    @Override
    public void onErrorSelfie(ErrorBio errorBio) { }
};

unicoCheckCamera.prepareSelfieCamera(new SelfieCameraListener() {
    @Override
    public void onCameraReady(UnicoCheckCameraOpener.Selfie cameraOpener) {
        cameraOpener.open(cameraListener);
    }

    @Override
    public void onCameraFailed(String message) {
        Log.e(TAG, message);
    }
});
```
  </TabItem>

  <TabItem value="kotlin" label="Kotlin">

```kotlin
val cameraListener: iAcessoBioSelfie = object : iAcessoBioSelfie {
    override fun onSuccessSelfie(result: ResultCamera?) {}

    override fun onErrorSelfie(errorBio: ErrorBio?) {}
}

unicoCheckCamera.prepareSelfieCamera(object : SelfieCameraListener {
    override fun onCameraReady(cameraOpener: UnicoCheckCameraOpener.Selfie?) {
        cameraOpener?.open(cameraListener)
    }

    override fun onCameraFailed(message: String?) {
        Log.e(TAG, message)
    }
})
```

  </TabItem>
</Tabs>


:::note
 O base64 retornado possui dados com informações e que deverão ser passados inteiramente via integração com o **unico check**.
:::


Caso queira converter o base64 para Bitmap, a maneira padrão não funcionará para o Android, sendo necessário realizar o split a partir da vírgula(,) para funcionamento correto. Caso queira saber mais, sugerimos o seguinte artigo [How to convert a Base64 string into a Bitmap image to show it in a ImageView?](https://stackoverflow.com/questions/4837110/how-to-convert-a-base64-string-into-a-bitmap-image-to-show-it-in-a-imageview)

</li>

<li>

### Chamar nossas APIs

A captura das imagens é apenas a primeira parte da nossa jornada. Após a capturar, você deverá enviar o `base64` gerado para nossas APIs, selecionando um dos fluxos disponíveis (detalhados [neste artigo](#)). Exemplo abaixo:

```bash
curl --location --request POST 'https://example.com/services/v3/AcessoService.svc/processes' \
--header 'APIKEY: 11111111-1111-1111-1111-111111111111' \
--header 'Authorization: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' \
--header 'Content-Type: application/json' \
--data-raw '{
  "subject": {
    "Code": "12345678910",
    "Name": "Bob",
    "Gender": "M",
    "BirthDate": "01/01/0001",
    "Email": "email@example.com",
    "Phone": "5543999999999"
  },
  "onlySelfie": true,
  "imagebase64": "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAAgSURBVDhPY/wPBAwUACYoTTYYNWDUABAYNWDgDWBgAABrygQclUTopgAAAABJRU5ErkJggg=="
}'

```


</li>

<li>

### Juntando todas as peças

Se você chegou até aqui, já deve ter tudo configurado e ready-to-go! Porém, não custa nada deixar dois exemplos de ponta-a-ponta.

Abaixo dois exemplos completos, com captura manual ou automática.

Como mencionado acima, em ambos os casos, caso o evento success seja disparado, iremos retornar um `base64` que deverá ser enviado para nossas APIs do motor biométrico.

```javascript
{
    base64: string
}
```

</li>
</ol>
</Steps>

<li>

### Configurar layout do frame (Opcional)

</li>




## Ficou com dúvidas?

Esperamos ter ajudado com este artigo. Não encontrou algo ou ainda precisa de ajuda? Disponibilizamos as seguintes opções para que você possa obter ajuda:

- Entre em contato através de nosso e-mail [suporte.unicocheck@unico.io](mailto:suporte.unicocheck@unico.io);
- Caso já seja um parceiro, você também pode entrar em contato através de nossa [Central de Ajuda](https://ajuda.unico.io/hc/pt-br/categories/360002344171);

## Próximos passos

Ótimo! Você chegou até aqui =). A seguir vamos te contar um pouco mais sobre nossa API ou sobre nossa funcionalidade de captura de documentos.

- [Guia para implantação de captura de documentos](reconhecimento-facial);
- [API Reference do SDK](API);















