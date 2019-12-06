  	        // create a redirect URI using an available port on the loopback address.
            string redirectUri = string.Format("http://localhost:5000/signin-oidc/");

            // create an HttpListener to listen for requests on that redirect URI.
            var http = new HttpListener();
            http.Prefixes.Add(redirectUri);

            //Start listen
            http.Start();

            //set options for the OAuth2 Connection (Make these as parameters)
            var options = new OidcClientOptions
            {
                Authority = "https://identity.xero.com",
                ClientId = "",
                ClientSecret = "",
                Scope = "offline_access openid profile email accounting.settings accounting.transactions",
                RedirectUri = redirectUri
            };
            options.Policy.Discovery.ValidateEndpoints = false;

            var client = new OidcClient(options);
            var state = await client.PrepareLoginAsync();

            // open system browser to start authentication
            Process.Start(state.StartUrl);

            // wait for the authorization response.
            var context = await http.GetContextAsync();

            var result = await client.ProcessResponseAsync(context.Request.Url.AbsoluteUri, state);

            if (!result.IsError)
            {
                Console.WriteLine("\n\nClaims:");
                foreach (var claim in result.User.Claims)
                {
                    Console.WriteLine("{0}: {1}", claim.Type, claim.Value);
                }

                Console.WriteLine();
                Console.WriteLine("Access token:\n{0}", result.AccessToken);

                if (!string.IsNullOrWhiteSpace(result.RefreshToken))
                {
                    Console.WriteLine("Refresh token:\n{0}", result.RefreshToken);
                }
            }
            else
            {
                Console.WriteLine("\n\nError:\n{0}", result.Error);
            }           

            var responseString = string.Format("All Done close down your browser");
            byte[] buffer = Encoding.UTF8.GetBytes(responseString);

            // and send it
            var response = context.Response;
            response.ContentType = "text/html";
            response.ContentLength64 = buffer.Length;
            response.StatusCode = 200;
            response.OutputStream.Write(buffer, 0, buffer.Length);
            response.OutputStream.Close();

            http.Stop();		
			
			var AccountingApi = new AccountingApi();

            try
            {
                //Now connect to Accounting API and get invoices
				        //Parameters Access Token and TennantID
                var test = await AccountingApi.GetInvoicesAsync(result.AccessToken, "");
                MessageBox.Show("Number of Invoices: " + test._Invoices.Count.ToString()0;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }

//If you save the result.AccessToken, result.RefreshToken for the use then when it expires, you can auto renew the Authentication

           // create a redirect URI using an available port on the loopback address.
            string redirectUri = string.Format("http://localhost:5000/signin-oidc/");
            Console.WriteLine("redirect URI: " + redirectUri);

            // create an HttpListener to listen for requests on that redirect URI.
            //var http = new HttpListener();
            //http.Prefixes.Add(redirectUri);
            //Console.WriteLine("Listening..");

            //Start listen
            //http.Start();

            //set options for the OAuth2 Connection (Make these as parameters)
            var options = new OidcClientOptions
            {
                Authority = "https://identity.xero.com",
                ClientId = "",
                ClientSecret = "",
                Scope = "offline_access openid profile email accounting.settings accounting.transactions",
                RedirectUri = redirectUri
            };
            options.Policy.Discovery.ValidateEndpoints = false;

            var client = new OidcClient(options);
            var state = await client.PrepareLoginAsync();

            Console.WriteLine($"Start URL: {state.StartUrl}");

            //refresh is the value saved from the initial authorisation from result.RefreshToken
            var result = await client.RefreshTokenAsync(refresh);

            if (!result.IsError)
            {
                //New Values after refresh
                tokencode = result.AccessToken;
                refresh = result.RefreshToken;
                expires = result.AccessTokenExpiration.ToString();
            }

