# TronNetCore3.1

fork from https://github.com/stoway/TronNet

Rebuild to .netcore 3.1

Code Sample:


1. Create Wallet


```cs


        public static void GenerateWalletOfflice()
        {

            var key = TronECKey.GenerateKey(TronNetwork.MainNet);

            var address = key.GetPublicAddress();

            Console.WriteLine(address);
            
        }


```

2.Create TRX tracation.


```cs



            var walletPrivateKey = "8ca224fec50afaffc70e445ae2d01434e666268298548f67597068735562ad10";



            var ecKey = new TronECKey(walletPrivateKey, TronNetwork.MainNet);
            var from = ecKey.GetPublicAddress();
            //收款錢包
            var to = "TRza1rbWRhgnRKvTSxnkzkz7RP1Xem6TFJ";


            //打出2顆 trx
            var amount = 2 * 1_000_000L;

            IServiceCollection services = new ServiceCollection();
            services.AddTronNet(x =>
            {
                x.Network = TronNetwork.MainNet;
                x.Channel = new GrpcChannelOption { Host = "47.252.19.181", Port = 50051 };
                x.SolidityChannel = new GrpcChannelOption { Host = "47.252.19.181", Port = 50052 };
                //原本以為要填寫，似乎不填寫也可以用
                x.ApiKey = "apikey";
            });

            services.AddLogging();
            var service = services.BuildServiceProvider();
            var transactionClient = service.GetService<ITransactionClient>();
            var transactionExtension = transactionClient.CreateTransactionAsync(from, to, amount).Result;

            var transactionSigned = transactionClient.GetTransactionSign(transactionExtension.Transaction, walletPrivateKey);

            //取得 sign
            var signed = JsonConvert.SerializeObject(transactionSigned);

            Console.WriteLine("-SIGN-");

            Console.WriteLine(signed);
            Console.WriteLine("-TXID-");
            Console.WriteLine(transactionSigned.GetTxid());

            //送出
            var result = transactionClient.BroadcastTransactionAsync(transactionSigned).Result;
            Console.WriteLine("-RESULT-");
            Console.WriteLine(JsonConvert.SerializeObject(result));



```


3.USDC transation


```cs

        /// <summary>
        /// Transfer USDC
        /// </summary>
        public static void TestContractTransation()
        {

            //發射端的私鑰
            //reference: https://blog.no2don.com/2021/11/c-trx-eth-nethereum.html
            var walletPrivateKey = "6984206e9e1af8090a120774fd657d88b278f8bde685ef8532dabe9d61f557bc";

            IServiceCollection services = new ServiceCollection();
            services.AddTronNet(x =>
            {
                x.Network = TronNetwork.MainNet;
                x.Channel = new GrpcChannelOption { Host = "47.252.19.181", Port = 50051 };
                x.SolidityChannel = new GrpcChannelOption { Host = "47.252.19.181", Port = 50052 };
                x.ApiKey = "apikey";
            });


            services.AddLogging();

            var service = services.BuildServiceProvider();
            var walletClient = service.GetService<IWalletClient>();

            var account = walletClient.GetAccount(walletPrivateKey);

            //USDT TOKEN
            var contractAddress = "TXLAQ63Xg1NAzckPwKHvzw7CSEmLMEqcdj";

            //收的錢包
            var to = "TLhTDxNcm9XaSq35mghq6Az35LeHdcn77S";

            var amount = 99;
            //手續費 10 TRX
            var feeAmount = 10 * 1_000_000L;

            var contractClientFactory = service.GetService<IContractClientFactory>();

            var contractClient = contractClientFactory.CreateClient(ContractProtocol.TRC20);

            //備註只能用英文
            var result = contractClient.TransferAsync(contractAddress, account, to, amount, "DONMA BLOG TEST", feeAmount).Result;

            Console.WriteLine("-- RESULT --");
            Console.WriteLine(JsonConvert.SerializeObject(result));

        }


```

