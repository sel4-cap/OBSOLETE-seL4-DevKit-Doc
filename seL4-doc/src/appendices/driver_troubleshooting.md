# Library Extension - Troubleshooting

_Expect we need to cover to following:_
- Known problem: DMA
- Known problem: Live tree
- Compile with verbose logging.
- May need to add new initialisation code. Look in u-boot’s common/board_r.c file to see if any is required.
- Environment variables not having expected effect - check for callbacks.
- Could flesh out some more details on the API in the 'usage' section, e.g. API to read characters from the U-Boot 'stdin' device and gt/put Ethernet frames.
