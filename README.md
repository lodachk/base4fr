# base4frimport time
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

# Uniswap V3 Swap event topic:
# keccak256("Swap(address,address,int256,int256,uint160,uint128,int24)")
SWAP_TOPIC = "0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67"


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))
    if not w3.is_connected():
        raise RuntimeError("❌ Cannot connect to Base RPC")

    print("✅ Connected to Base")
    last = w3.eth.block_number
    print("Starting block:", last)

    while True:
        try:
            current = w3.eth.block_number

            if current > last:
                for b in range(last + 1, current + 1):
                    logs = w3.eth.get_logs(
                        {
                            "fromBlock": b,
                            "toBlock": b,
                            "topics": [SWAP_TOPIC],
                        }
                    )

                    if logs:
                        print(f"\nBlock {b} | UniswapV3 swaps: {len(logs)}")

                    for log in logs[:50]:
                        pool = log["address"]
                        tx = log["transactionHash"].hex()

                        # We are not decoding amounts here (keeps repo tiny).
                        # This script is mainly for detecting swaps + pool addresses.
                        print(f"Swap | pool={pool} | tx={tx}")

                last = current

            time.sleep(2)

        except KeyboardInterrupt:
            print("\nStopped.")
            return

        except Exception as e:
            print("Error:", str(e))
            time.sleep(3)


if __name__ == "__main__":
    main()
