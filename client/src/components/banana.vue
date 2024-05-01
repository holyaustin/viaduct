<script setup lang="ts">

import "@rainbow-me/rainbowkit/styles.css";
//import Demo from "./Demo";
import {RainbowKitProvider, connectorsForWallets } from "@rainbow-me/rainbowkit";
import {
  metaMaskWallet,
  injectedWallet,
  rainbowWallet,
  walletConnectWallet
} from "@rainbow-me/rainbowkit/wallets";

import { polygonMumbai, optimismGoerli, goerli } from "@wagmi/chains";
import { publicProvider } from "wagmi/providers/public";

import { configureChains, createClient, WagmiConfig } from "wagmi";
import { BananaWallet } from '@rize-labs/banana-rainbowkit-plugin'


  const { chains, provider } = configureChains(
    // currently on these three chains are supported by BananaWallet
    [polygonMumbai, optimismGoerli, goerli],
    [publicProvider()]
  );

  const connectors = connectorsForWallets([
    {
      groupName: "Recommended",
      wallets: [
       BananaWallet({ chains, connect: { networkId: 80001 } }),
       metaMaskWallet({ chains, shimDisconnect: true }),
       rainbowWallet({ chains }),
       walletConnectWallet({ chains }),
        injectedWallet({ chains, shimDisconnect: true }),
      ]
    },
  ]);

  const wagmiClient = createClient({
    autoConnect: true,
    connectors,
    provider,
  });

  </script>

<template>
    <div className="App">
      <WagmiConfig client={wagmiClient}>
        <RainbowKitProvider chains={chains}>
          <Demo />
        </RainbowKitProvider>
      </WagmiConfig>
    </div>
</template>

