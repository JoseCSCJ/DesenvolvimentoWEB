# Injeção de Dependências
## Asp.Net Core 

A injeção de dependências é um *design pattern* no qual uma classe ou objeto é passada por outra classe ou objeto, ao invés do padrão de criá-los diretamente. Se ao efetuar uma ação, uma classe precisa de colher informação de outra classe, essa é uma *dependência*, que pode ser implicita ou explicita.</br>

Uma **dependência explicita** aparece no construtor de uma classe ou em uma lista de parâmetros em um método, no caso de dependências mais locais.</br>
Uma **dependência implicita** se existe somente no código dentro da classe e não na sua interface publica.</br>

A dependência explícita tem maiores vantagens em relação à implícita por ter maior capacidade de manutenção e clareza. Trabalhar com uma dependência implicita exige sintonia com o seu código para ser capaz de localizar as devidas dependências. 

### Implicita x Explicita
#### Exemplo - Dependência Implicita
##### Retirado de [Deviq]
```
using System;
using System.IO;
using System.Linq;

namespace ImplicitDependencies
{
    class Program
    {
        static void Main(string[] args)
        {
            var customer = new Customer()
            {
                FavoriteColor = "Blue",
                Title = "Mr.",
                Fullname = "Steve Smith"
            };
            Context.CurrentCustomer = customer;

            var response = new PersonalizedResponse();

            Console.WriteLine(response.GetResponse());
            Console.ReadLine();
        }
    }

    public static class Context
    {
        public static Customer CurrentCustomer { get; set; }

        public static void Log(string message)
        {
            using (StreamWriter logFile = new StreamWriter(
                Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
                    "logfile.txt")))
            {
                logFile.WriteLine(message);
            }
        }
    }

    public class Customer
    {
        public string FavoriteColor { get; set; }
        public string Title { get; set; }
        public string Fullname { get; set; }
    }

    public class PersonalizedResponse
    {
        public string GetResponse()
        {
            Context.Log("Generating personalized response.");
            string formatString = "Good {0}, {1} {2}! Would you like a {3} widget today?";
            string timeOfDay = "afternoon";
            if (DateTime.Now.Hour < 12)
            {
                timeOfDay = "morning";
            }
            if (DateTime.Now.Hour > 17)
            {
                timeOfDay = "evening";
            }
            return String.Format(formatString, timeOfDay, 
                Context.CurrentCustomer.Title, 
                Context.CurrentCustomer.Fullname, 
                Context.CurrentCustomer.FavoriteColor);
        }
    }
}

```

#### Exemplo - Dependência Explicita
##### Retirado de [Deviq]
```
using System;
using System.IO;
using System.Linq;

namespace ExplicitDependencies
{
    class Program
    {
        static void Main(string[] args)
        {
            var customer = new Customer()
            {
                FavoriteColor = "Blue",
                Title = "Mr.",
                Fullname = "Steve Smith"
            };

            var response = new PersonalizedResponse(new SimpleFileLogger(), new SystemDateTime());

            Console.WriteLine(response.GetResponse(customer));
            Console.ReadLine();
        }
    }

    public interface ILogger
    {
        void Log(string message);
    }

    public class SimpleFileLogger : ILogger
    {
        public void Log(string message)
        {
            using (StreamWriter logFile = new StreamWriter(
                Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
                    "logfile.txt")))
            {
                logFile.WriteLine(message);
            }
        }
    }

    public interface IDateTime
    {
        DateTime Now { get; }
    }

    public class SystemDateTime : IDateTime
    {
        public DateTime Now
        {
            get
            {
                return DateTime.Now;
            }
        }
    }

    public class Customer
    {
        public string FavoriteColor { get; set; }
        public string Title { get; set; }
        public string Fullname { get; set; }
    }

    public class PersonalizedResponse
    {
        private readonly ILogger _logger;

        private readonly IDateTime _dateTime;

        public PersonalizedResponse(ILogger logger,
            IDateTime dateTime)
        {
            this._dateTime = dateTime;
            this._logger = logger;
        }

        public string GetResponse(Customer customer)
        {
            _logger.Log("Generating personalized response.");
            string formatString = "Good {0}, {1} {2}! Would you like a {3} widget today?";
            string timeOfDay = "afternoon";
            if (_dateTime.Now.Hour < 12)
            {
                timeOfDay = "morning";
            }
            if (_dateTime.Now.Hour > 17)
            {
                timeOfDay = "evening";
            }
            return String.Format(formatString, timeOfDay,
                customer.Title,
                customer.Fullname,
                customer.FavoriteColor);
        }
    }
}
``` 
Ao comparar ambos os casos é notável a diferença no uso das **interfaces** no caso de dependências explícitas.</br> 

</br>

A injeção de dependências é configurada no Startup.cs através de serviços, deixando-os disponíveis para a aplicação. O *lifetime* de uma injeção de dependências especifica quando um objeto é criado ou recriado, existindo três opções:

- Singleton: Uma instância será criada e compartilhada para todos
- Scoped: Uma instância por *request* da aplicação será criada
- Transient: Instâncias não serão compartilhadas e serão criadas cada vez que requeridas.



## Referências
[Microsoft]https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/dependency-injection
[Deviq]http://deviq.com/explicit-dependencies-principle/
[InforWorld]https://www.infoworld.com/article/3232636/application-development/how-to-use-dependency-injection-in-aspnet-core.html

