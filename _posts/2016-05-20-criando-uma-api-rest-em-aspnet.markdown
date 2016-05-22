---
layout: post
title:  "Criando uma api rest em ASP.NET"
date:   2016-05-20 22:48:05 -0300
categories: dev
---
# **Introdução**

Nest blog post você irá aprender como criar e consumir um RESTful Web Service usando o framework ASP.NET e linguagem C#.

# **Instalação**

Primeiramente, você deve baixar as ferramentas necessárias para desenvolvimento.

Caso você seja um usuário Windows, pode baixar uma versão do Visual Studio. Dentre as distribuições, o Visual Studio Comunnity é de graça e serve todos os propósitos para este exemplo. Você pode baixar ele [aqui](https://www.visualstudio.com/products/visual-studio-community-vs "Visual Studio Community"){:target="_blank"}.

Caso você seja um usuário Mac ou de alguma distribuição Linux, pode utilizar o MonoDevelop. Este encontra-se [aqui](http://www.monodevelop.com/download/ "Visual Studio Community"){:target="_blank"}.

# **Criando um projeto**

Vamos seguir os passos com o Visual Studio, mas os conceitos e os passos podem ser aplicados da mesma forma para o MonoDevelop.

Uma vez que tenha instalado, você verá a página incial.
![VS Studio](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_1.png)

Agora, vá em File>New>Project.. ou aperte Ctrl+Shift+N
![VS Studio](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_2.png)

A tela de criação de um projeto será aberta e você terá várias opções de projeto para selecionar. Vá em Templates>Visual C#>Web e escolha a opção ASP.NET Web Application.
![VS Studio](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_3.png)

Após isso escolha a opção Web API já que queremos desenvolver um serviço REST. Como boa prática selecione a opção "Add unit tests" para incluir um projeto de testes unitários, dessa forma você pode desenvolver testes automáticos para sua aplicação.
![VS Studio](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_4.png)

Depois de finalizado você terá uma tela parecida com esta.
![VS Studio](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_5.png)

Podemos notar que o template segue o padrão de [MVC](https://pt.wikipedia.org/wiki/MVC "MVC"){:target="_blank"}. Dessa forma o pacote Model contem as classes que representam os modelos, o pacote controller contem as classes que mediam a entrada, convertendo-a em comandos para o modelo ou para a view, o pacote view apresenta os dados.

# **Aplicação**

Para este exemplo vamos escrever a boa e velha calculadora :D. Mas para deixarmos as coisas um pouco mais interessantes vamos usar [Padrões de Projeto](https://pt.wikipedia.org/wiki/Padr%C3%A3o_de_projeto_de_software "Padrões de Projeto Wikipedia"){:target="_blank"}. 

Para isto, ao invés de criarmos uma única classe **Calculator** que possua 4 métodos para as operações básicas(soma, subtração, multipicação e divisão) vamos usar um [Padrão de Estratégia](https://pt.wikipedia.org/wiki/Strategy "Padrão de EstratégiaWikipedia"){:target="_blank"}.

Primeiramente, crie a interface da calculadora que provê um método que representa uma operação.
{% highlight csharp %}
public interface ICalculate 
{  
    int Calculate(int value1, int value2);
}
{% endhighlight %}

Adicione uma classe Plus que implementa esta interface e faz a operação ser uma soma.
{% highlight csharp %}
public class Plus : ICalculate 
{
    public int Calculate(int value1, int value2) 
    {
        return value1 + value2;
    }
}
{% endhighlight %}

Adicione uma classe Plus que implementa esta interface e faz a operação ser uma subtração.
{% highlight csharp %}
public class Minus : ICalculate 
{
    public int Calculate(int value1, int value2) 
    {
        return value1 - value2;
    }
}
{% endhighlight %}

Por fim, adicione uma classe Calculator que contem um atributo e um método calculate que irá fazer um binding em tempo de execução para o método da classe(Plus ou Minus) que for passada no construtor quando esta classe for criada.
{% highlight csharp %}
public class Calculator
{
    private ICalculate calculateStrategy;

    public Calculator(ICalculate strategy) 
    {
    	calculateStrategy = strategy;
    }

    public int Calculate(int value1, int value2) 
    {
    	return calculateStrategy.Calculate(value1, value2);
    }
}
{% endhighlight %}

# **Web Service**

Na pasta Controllers clique com o botão diretiro e selecione **add controler** e escolha a opção **Web API Controller - Empty**. Nomeie esta classe como CalculatorController. Esta classe ira definir o recurso a ser utilizado e a lógica que será implementada.

{% highlight csharp %}
using CalculatorExample.Business;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;

namespace CalculatorExample.Controllers
{
    public class CalculatorController : ApiController
    {
        [Route("api/calculator/sum")]
        [HttpGet]
        public int Sum([FromUri]int value1, [FromUri]int value2)
        {
            Calculator calculator = new Calculator(new Plus());

            return calculator.Calculate(value1, value2);
        }

        [Route("api/calculator/minus")]
        [HttpGet]
        public int Minus([FromUri]int value1, [FromUri]int value2)
        {
            Calculator calculator = new Calculator(new Minus());

            return calculator.Calculate(value1, value2);
        }
    }
}
{% endhighlight %}

A anotação **[Route]** especifíca a rota para chegar a este recurso. **[HttpGet]** especifíca o mêtodo a ser usado, neste caso GET, **[FromUri]** faz um parse das variáveis que são passadas na url, neste caso value1 e value2.

Assim, poderiamos acessar estes métodos como feito nas imagens a seguir:

Soma
![Sum](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_6.png)

Subtração
![Minus](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_7.png)

# **Cliente**

Para nosso cliente, vamos alterar o arquivo **Index.cshtml** localizado na pasta **Views/Home**. Para este exemplo, podemos utilizar dois campos de texto e dois botões(um de soma e outro de subtração) e um campo para o resultado. Quando um dos botões é clicado a chamada é feita ao serviço e o resultado é mostrado.

{% highlight html %}
<div id="body" class="jumbotron">
    <input type="text" id="value1">
    <input type="text" id="value2">
    <button onclick="Sum()">Soma</button>
    <button onclick="Minus()">Subtrai</button>
    <br />
    Resultado: <label id="result"></label>
</div>

@section scripts{
    <script type="text/javascript">
        function Sum() {
            $.getJSON("/api/calculator/sum/?value1=" + $('#value1').val() +
                "&value2=" + $('#value2').val(), function (data)
            {
                $(data).each(function(i, item)
                {
                    $('#result').text(item);
                });
            });
        }
        
        function Minus() {
            $.getJSON("/api/calculator/minus/?value1=" + $('#value1').val() +
                "&value2=" + $('#value2').val(), function (data) {
                    $(data).each(function (i, item) {
                        $('#result').text(item);
                    });
                });
        }
    </script>
}
{% endhighlight %}

Caso tenha seguido todos os passo a passo você terá como resultado final algo como
![App](/assets/2016-05-20-criando-uma-api-rest-em-aspnet/image_8.png)

Dê uma olhada no [repositório][calculator-github]{:target="_blank"} para uma versão deste exemplo.

[calculator-github]: https://github.com/CarlosRodrigo/calculator-webservice-example