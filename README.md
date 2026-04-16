# TronNetCore3.1

TronNet 的 .NET Core 3.1 版本。

這個專案是從原始 `TronNet` fork 後整理重建，目標是讓需要在 `.NET Core 3.1` 環境中操作 TRON 鏈的開發者，可以直接使用熟悉的 C# 方式完成常見功能，例如：

- 建立錢包
- 建立 TRX 轉帳交易
- 發送 TRC20 Token 交易

---

## Project Goal

有些既有專案仍停留在 `.NET Core 3.1`，但又有串接 TRON 鏈、TRX、USDT(TRC20) 這類需求。

這個專案的定位很直接，就是把 TronNet 整理成可以在 `.NET Core 3.1` 上使用的版本，讓舊系統或既有服務不需要整個升級框架，也能先接上 TRON 相關功能。

---

## Features

- 支援 `.NET Core 3.1`
- 建立 TRON Wallet
- 建立 TRX 轉帳交易
- 支援 TRC20 Token 轉帳
- 使用 C# 與 DI 方式整合服務
- 適合作為 TRON 串接測試或基礎專案

---

## Repository Structure

目前 repo 內主要包含：

- `src`
- `TestTronNet`
- `TronNet.sln`

---

## Code Samples

## 1. Create Wallet

~~~csharp
public static void GenerateWalletOfflice()
{
    var key = TronECKey.GenerateKey(TronNetwork.MainNet);
    var address = key.GetPublicAddress();

    Console.WriteLine(address);
}
~~~

---

## 2. Create TRX Transaction

~~~csharp
var walletPrivateKey = "your-private-key";

var ecKey = new TronECKey(walletPrivateKey, TronNetwork.MainNet);
var from = ecKey.GetPublicAddress();
var to = "receiver-wallet-address";

// 2 TRX
var amount = 2 * 1_000_000L;

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
var transactionClient = service.GetService<ITransactionClient>();
var transactionExtension = transactionClient.CreateTransactionAsync(from, to, amount).Result;

var transactionSigned = transactionClient.GetTransactionSign(
    transactionExtension.Transaction,
    walletPrivateKey
);

Console.WriteLine(transactionSigned.GetTxid());

var result = transactionClient.BroadcastTransactionAsync(transactionSigned).Result;
Console.WriteLine(JsonConvert.SerializeObject(result));
~~~

---

## 3. TRC20 Transaction

~~~csharp
public static void TestContractTransation()
{
    var walletPrivateKey = "your-private-key";

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

    // Example: USDT contract on TRON
    var contractAddress = "TXLAQ63Xg1NAzckPwKHvzw7CSEmLMEqcdj";

    var to = "receiver-wallet-address";
    var amount = 99;
    var feeAmount = 10 * 1_000_000L;

    var contractClientFactory = service.GetService<IContractClientFactory>();
    var contractClient = contractClientFactory.CreateClient(ContractProtocol.TRC20);

    var result = contractClient.TransferAsync(
        contractAddress,
        account,
        to,
        amount,
        "DONMA BLOG TEST",
        feeAmount
    ).Result;

    Console.WriteLine(JsonConvert.SerializeObject(result));
}
~~~

---

## Getting Started

### 1. Clone

~~~bash
git clone https://github.com/donma/TronNetCore3.1.git
~~~

### 2. Open Solution

使用 Visual Studio 或其他支援 .NET 的 IDE 開啟：

~~~text
TronNet.sln
~~~

### 3. Build and Run

~~~bash
dotnet restore
dotnet build
~~~

如果要直接看範例程式，可優先從 `TestTronNet` 開始。

---

## Notes

- 本專案是 `TronNet` 的 `.NET Core 3.1` 重建版本
- 比較適合既有舊系統或維護中的 .NET Core 3.1 專案
- README 範例包含主網 gRPC 節點設定
- 實際上線前，建議自行確認節點可用性、手續費設定、合約地址與網路環境
- 私鑰請勿直接硬編碼在正式系統中

---

## License

本專案採用 MIT 授權。

版權所有 (c) 2021 Donma

茲授權任何取得本軟體及相關文件副本之人，得不受限制地使用本軟體，包括但不限於使用、複製、修改、合併、出版、散布、再授權及販售本軟體之副本，並允許接受本軟體之人為上述行為，但須符合下列條件：

上述版權聲明及本授權聲明，應包含於本軟體之所有副本或重要部分中。

本軟體係依現況提供，不附任何明示或默示之保證，包括但不限於適售性、特定用途適用性及不侵權之保證。在任何情況下，無論係因契約行為、侵權行為或其他原因所生之任何請求、損害或其他責任，著作權人或版權所有人均不負任何責任，即使該等責任係因本軟體或本軟體之使用或其他交易而起。

---

## 免責聲明

本專案主要作為開發、測試與技術研究用途。

區塊鏈交易、私鑰管理、TRC20 合約操作與鏈上廣播，都具有一定風險。使用者在正式環境中使用本專案前，應自行驗證交易流程、節點可用性、合約地址、費用設定與安全性設計。

因使用本專案所造成之任何直接或間接損失、交易失敗、資產損失、資料問題或其他責任，專案作者與貢獻者均不負擔任何責任。
