#YACLP
##Yet Another Command-Line Parser

It's on NuGet:

    Install-Package YACLP

Why another one?

Because there were a bunch out there but all of them focused more on the parsing than on being quick and easy to call.

I want my command-line parser to not only parse arguments (which it does, but isn't very flexible about) but to automatically generate a usage message so that I don't have to.

##Simple Usage
    
    var options = DefaultParser.ParseOrExitWithUsageMessage<MyCommandLineParameters>(args);

##Recommendations
I'd recommend using an IConfiguration or similar interface so that anything that depends on it doesn't need to know about command-line parameters.

Our main program would look like this:

    public class Program
    {
        private static void Main(string[] args)
        {
            var configuration = DefaultParser.ParseOrExitWithUsageMessage<CommandLineParameters>(args);

            new Greeter(configuration).SayHello();
        }
    }

... and our Greeter like this:

    public class Greeter
    {
        private readonly IConfiguration _configuration;

        public Greeter(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        public void SayHello()
        {
            var message = string.IsNullOrEmpty(_configuration.Surname)
                              ? string.Format("Hi, {0}!", _configuration.FirstName)
                              : string.Format("Hello, Mr/Ms {0}", _configuration.Surname);

            Console.WriteLine(message);
        }
    }

Note that our Greeter takes a dependency on an IConfiguration, which looks like this:

    public interface IConfiguration
    {
        string FirstName { get; set; }
        string Surname { get; set; }
    }

... and that IConfiguration interface is implemented by our CommandLineParameters class:

    public class CommandLineParameters : IConfiguration
    {
        [ParameterDescription("The first name of the person using the program.")]
        public string FirstName { get; set; }

        [ParameterIsOptional]
        [ParameterDefault("Smith")]
        [ParameterDescription("The last name of the person using the program.")]
        public string Surname { get; set; }
    }

The key point here is that our Greeter knows absolutely nothing about command-line parameters as everything is segregated using the IConfiguration interface.