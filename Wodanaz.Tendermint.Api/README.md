## Tendermint C# GRPC ABCI Assembly

This is a precompiled package of the Tendermint ABCI API. The API is also mostly compatible with the current [CosmosBFT](https://github.com/cometbft/cometbft) 0.38 implementation.

#### Tendermint Server

Implementing a Tendermint Server is quite easy. Preprare your ASP.NET core Web Application or API Project for gRPC use and implement the abstract ABCIApplication.ABCIApplicationBase class:

```
	public class WodanazApp : ABCIApplication.ABCIApplicationBase, IWodanazApp
	{
		private IConfiguration _conf;

		public WodanazApp(IConfiguration conf)
		{
			_conf = conf;
		}

		public override Task<ResponseInfo> Info(RequestInfo request, ServerCallContext context)
		{
			// Only fake results
			return Task.FromResult(new ResponseInfo() { AppVersion = 1, Version = "1.0", Data = "Data" });
		}

		public override Task<ResponseInitChain> InitChain(RequestInitChain request, ServerCallContext context)
		{
			// Fake result
			return Task.FromResult(new ResponseInitChain() { });
		}

		public override Task<ResponseEcho> Echo(RequestEcho request, ServerCallContext context)
		{
			// Some echo info
			var echo = new ResponseEcho { Message = $"Validator is Running: {DateTime.Now:dd-MM-yyyy HH:mm}" };
			return Task.FromResult(echo);
		}
	}
```

#### Testing the server

If you added the [Grpc.Tools]() package, you can simply use a generic client, e.g [grpcui](https://github.com/fullstorydev/grpcui) like this

```
grpcui -insecure 127.0.0.1:5001
```

![gRPC Test with grpcui](https://github.com/TheCyberCore/Wodanaz.Tendermint.Api/raw/master/grpcui_example_echo.png) 


Or you can install a local version of Tendermint as described [here](https://docs.tendermint.com/v0.34/introduction/install.html), initialize the node properly and then start tendermint like this:

```
tendermint node --abci grpc --proxy_app 127.0.0.1:5001
```

Note that .NET 6.0 by default publishes the gRPC endpoints on port 5000 (http) and 5001 (https). If you use different ports change the command accordingly.
In my tests, Tendermint was not able to do TLS handshakes with a .NET core HTTPS endpoint properly. This means that you have to restrict Kestrel to Http2 only connections. I managed to establish a connection between Tendermint and the C# app using the following Kestrel configuration.

```
{
  "AllowedHosts": "*",
  "Kestrel": {
    "Endpoints": {
      "gRPC": {
        "Protocols": "Http2",
        "Url": "http://127.0.0.1:5000"
      }
    }
  }
}
```
