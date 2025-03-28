<script lang="ts">
  import * as Card from "$lib/components/ui/card";
  import { Button } from "$lib/components/ui/button";
  import { Separator } from "$lib/components/ui/separator";
  import { ArrowRight } from "@lucide/svelte";

  interface Props {
    client: {
      client_id: string;
      name: string;
      description: string;
      hasLogo?: boolean;
      logoUrl?: string | null;
      icon?: string | null;
      logoError?: boolean;
      accessGroups?: string[];
      restrictedAccess?: boolean;
      callback_urls?: string[]; // Added to support callback URLs
    };
    index?: number; // Add index prop for staggered animation
    columns?: number; // Number of columns in the grid
  }

  let {
    client = $bindable(),
    index = $bindable(0),
    columns = $bindable(3),
  }: Props = $props();

  // Handle image error
  function handleImageError() {
    client.logoError = true;
  }

  // Calculate animation delay based on position in grid
  function calculateAnimationDelay(index: number, columns: number): string {
    const row = Math.floor(index / columns);
    const col = index % columns;

    // Combine row and column to get a diagonal wave effect
    // The higher the sum, the later the animation
    const delay = (row + col) * 60; // 60ms per step

    return `${delay}ms`;
  }

  const animationDelay = calculateAnimationDelay(index, columns);

  // Extract base URL from callback URL
  // For example: https://example.test.com/callback -> https://example.test.com
  function getBaseUrl(callbackUrls: string[] | undefined): string {
    // If no callback URLs are available, return a placeholder
    if (!callbackUrls || callbackUrls.length === 0) {
      console.warn(`No callback URLs found for client ${client.name}`);
      return "#"; // Return a hash to prevent navigation
    }

    try {
      // Take the first callback URL
      const callbackUrl = callbackUrls[0];

      if(callbackUrl.substring(0,5) === "LINK#") {
        return callbackUrl.substring(5);
      }

      // Parse the URL to extract just the origin (protocol + domain + port)
      const url = new URL(callbackUrl);
      return url.origin;
    } catch (error) {
      console.error(`Error parsing callback URL for ${client.name}:`, error);
      // Return a hash to prevent navigation
      return "#";
    }
  }

  // Generate a consistent background gradient based on client name
  function generateGradient(name: string) {
    const hash = name
      .split("")
      .reduce((acc, char) => acc + char.charCodeAt(0), 0);
    const hue1 = hash % 360;
    const hue2 = (hue1 + 40) % 360;
    return `linear-gradient(135deg, hsla(${hue1}, 80%, 80%, 0.6), hsla(${hue2}, 80%, 60%, 0.7))`;
  }

  // Generate a subtle backdrop color for the logo based on client name
  function generateLogoBackdrop(name: string) {
    const hash = name
      .split("")
      .reduce((acc, char) => acc + char.charCodeAt(0), 0);
    const hue = hash % 360;
    return `hsla(${hue}, 70%, 97%, 0.8)`;
  }

  // Get the launch URL directly from the client's callback URLs
  let launchUrl = $state(getBaseUrl(client.callback_urls));
</script>

<Card.Root
  class="h-full flex flex-col justify-between overflow-hidden hover:shadow-lg transition-all hover:translate-y-[-2px] duration-300 border bg-card animate-fade-in"
  style="opacity: 0; transform: translateY(10px); animation-delay: {animationDelay}; animation-fill-mode: forwards;"
>
  <!-- Top colored bar with gradient -->
  <div
    class="h-3 w-full"
    style="background: {generateGradient(client.name)};"
  ></div>

  <div class="flex flex-col p-5 gap-4 flex-1">
    <!-- Logo and App Info -->
    <div class="flex items-start gap-4">
      <!-- Logo with backdrop -->
      <div
        class="rounded-xl flex-shrink-0 flex items-center justify-center w-16 h-16 p-2 border shadow-sm"
        style="background: {client.hasLogo && !client.logoError
          ? generateLogoBackdrop(client.name)
          : 'var(--primary-50, #f0f9ff)'};
               box-shadow: 0 2px 8px rgba(0,0,0,0.05), inset 0 0 0 1px rgba(255,255,255,0.5);"
      >
        {#if client.hasLogo && !client.logoError}
          <img
            src={client.logoUrl}
            alt="{client.name} logo"
            class="w-full h-full object-contain"
            loading="lazy"
            onerror={handleImageError}
          />
        {:else}
          <div
            class="w-full h-full flex items-center justify-center bg-gradient-to-br from-primary/10 to-primary/20 rounded-lg"
          >
            <span class="text-2xl">{client.icon || "📱"}</span>
          </div>
        {/if}
      </div>

      <!-- App Info -->
      <div class="flex-1 min-w-0">
        <Card.Title class="text-lg font-medium mb-1 line-clamp-2 break-words">
          {client.name}
        </Card.Title>
        {#if client.description}
          <p class="text-sm text-muted-foreground line-clamp-2">
            {client.description}
          </p>
        {/if}
      </div>
    </div>
  </div>

  <!-- Launch Button -->
  <div class="mt-auto">
    <Separator />
    <div class="p-3">
      <Button
        variant="default"
        class="w-full rounded-md font-medium"
        href={launchUrl}
        target="_blank"
        rel="noopener noreferrer"
        disabled={launchUrl === "#"}
      >
        <ArrowRight class="w-4 h-4 mr-2" />
        Launch App
      </Button>
    </div>
  </div>
</Card.Root>
