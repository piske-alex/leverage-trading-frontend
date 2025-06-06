<script lang="ts">
  import type { Position } from '$lib/stores/positions';
  import { positions } from '$lib/stores/positions';
  import { closePosition } from '$lib/services/trading';
  import { formatNumber } from '$lib/utils/formatters';
  import { TrendingUp, Loader2 } from 'lucide-svelte';
  import { formatPrice } from '$lib/services/birdeye';
  
  export let position: Position;
  
  let closing = false;
  
  async function handleClose() {
    closing = true;
    try {
      await closePosition(position.loanId);
      positions.updatePosition(position.id, { status: 'closed' });
    } catch (e) {
      console.error('Failed to close position:', e);
    } finally {
      closing = false;
    }
  }
  
  function formatDate(timestamp: number): string {
    return new Date(timestamp).toLocaleString('en-US', {
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    });
  }
</script>

<div class="card position-long hover:border-white/20 transition-all duration-200">
  <div class="flex items-start justify-between mb-4">
    <div class="flex items-center gap-3">
      <div class="p-2 bg-green-500/20 rounded-lg">
        <TrendingUp class="w-4 h-4 text-green-400" />
      </div>
      <div>
        <div class="flex items-center gap-2">
          <h3 class="font-semibold">{position.asset}</h3>
          <span class="leverage-badge">{position.leverage}x</span>
        </div>
        <p class="text-xs text-gray-400">{formatDate(position.timestamp)}</p>
      </div>
    </div>
    
    {#if position.status !== 'closed'}
      <button
        on:click={handleClose}
        disabled={closing}
        class="btn-secondary text-xs"
      >
        {#if closing}
          <Loader2 class="w-4 h-4 animate-spin" />
          <span class="sr-only">Closing...</span>
        {:else}
          Close
        {/if}
      </button>
    {/if}
  </div>
  
  <div class="grid grid-cols-2 gap-4 mb-4">
    <div>
      <p class="text-xs text-gray-400 mb-1">Entry Price</p>
      <p class="font-mono font-medium">{formatPrice(position.entryPrice)} BNB</p>
    </div>
    
    <div>
      <p class="text-xs text-gray-400 mb-1">{position.status == 'closed' ? 'Closing' : 'Current'} Price</p>
      <p class="font-mono font-medium">{formatPrice(position.status == 'closed' ? position.closingPositionSize / position.baseAmount : position.currentPrice)} BNB</p>
    </div>
    
    <div>
      <p class="text-xs text-gray-400 mb-1">Position Size</p>
      <p class="font-mono font-medium">{formatNumber(position.size, 4)} {position.asset}</p>
    </div>
    
    <div>
      <p class="text-xs text-gray-400 mb-1">Collateral</p>
      <p class="font-mono font-medium">{formatNumber(position.collateral, 4)} BNB</p>
    </div>
  </div>
  
  <div class="pt-4 border-t border-white/10">
    <div class="flex items-center justify-between">
      <div>
        <p class="text-xs text-gray-400 mb-1">PnL</p>
        <p class="font-mono text-lg font-semibold {position.pnl >= 0 ? 'text-green-400' : 'text-red-400'}">
          {position.pnl >= 0 ? '+' : ''}{formatNumber(position.pnl, 4)} BNB
        </p>
      </div>
      
      <div class="text-right">
        <p class="text-xs text-gray-400 mb-1">ROI</p>
        <p class="font-mono text-lg font-semibold {position.pnlPercentage >= 0 ? 'text-green-400' : 'text-red-400'}">
          {position.pnlPercentage >= 0 ? '+' : ''}{formatNumber(position.pnlPercentage, 2)}%
        </p>
      </div>
    </div>
    
    <div class="mt-3 p-2 bg-orange-500/10 border border-orange-500/20 rounded-lg">
      <p class="text-xs text-orange-400">
        Liquidation: ${formatPrice(position.liquidationPrice)}
      </p>
    </div>
  </div>
</div>