<script lang="ts">
  import { auth } from "$lib/stores/auth.store";
  import ClientCard from "$lib/components/ClientCard.svelte";
  import * as Pagination from "$lib/components/ui/pagination/index.js";
  import { Badge } from "$lib/components/ui/badge";
  import type { Client, PageServerData, UserGroup } from "$lib/types";
  import { onMount } from "svelte";
  import { Users, Search, LayoutDashboard, Filter, Zap } from "@lucide/svelte";

  interface Props {
    data: PageServerData;
  }

  let { data }: Props = $props();

  // All data now comes from server
  let userGroups: UserGroup[] = data.userGroups || [];
  const accessibleClients: Client[] = data.clients?.data || [];
  const error = data.status === "error" ? data.error : null;

  // Pagination state
  let itemsPerPage = 6; // 3 rows of 3 cards in desktop view

  // Calculate total items
  let totalItems = $derived(accessibleClients.length);

  // Get current page items based on pagination component's current page
  function getPaginatedClients(currentPage: number) {
    return accessibleClients.slice(
      (currentPage - 1) * itemsPerPage,
      currentPage * itemsPerPage
    );
  }

  // Default to 3 columns, but we'll update this based on screen size
  let gridColumns = $state(4);
  let searchTerm = $state("");

  onMount(() => {
    // Update columns based on screen size
    updateGridColumns();

    // Listen for window resize to update columns
    window.addEventListener("resize", updateGridColumns);

    // Apply fade-in animation to the header
    const header = document.querySelector(".dashboard-header");
    if (header) {
      header.classList.add("animate-fade-in");
    }

    return () => {
      window.removeEventListener("resize", updateGridColumns);
    };
  });

  function updateGridColumns() {
    // Determine number of columns based on screen width
    if (window.innerWidth >= 1280) {
      gridColumns = 4; // xl screens
    } else if (window.innerWidth >= 1024) {
      gridColumns = 3; // lg screens
    } else if (window.innerWidth >= 768) {
      gridColumns = 2; // md screens
    } else {
      gridColumns = 1; // sm screens
    }
  }
</script>

<svelte:head>
  <title>Applications - fischers.cc Portal</title>
</svelte:head>

<!-- Modern Dashboard Layout -->
<div class="space-y-8 max-w-[1200px] mx-auto flex flex-col h-full">
  <!-- Hero Header Section with Gradient Background -->
  <div
    class="dashboard-header rounded-2xl bg-gradient-to-r from-primary/10 to-primary/5 p-8 border shadow-sm"
  >
    <div
      class="flex flex-col md:flex-row items-start md:items-center justify-between gap-6"
    >
      <div class="space-y-2">
        <div class="flex items-center gap-3">
          <div class="bg-primary/10 p-2 rounded-full">
            <LayoutDashboard class="h-6 w-6 text-primary" />
          </div>
          <h1 class="text-3xl font-bold tracking-tight">My Applications</h1>
        </div>
        <p class="text-muted-foreground max-w-lg">
          Welcome to your fischers.cc dashboard, <span class="font-black"
            >{$auth.user?.name || $auth.user?.email || "User"}!</span
          > Access all your authorized applications from this portal.
        </p>
      </div>

      <!-- Search Bar -->
      <div class="relative w-full md:w-auto">
        <div
          class="flex items-center h-10 w-full md:w-[300px] rounded-xl border bg-background px-3 focus-within:ring-1 focus-within:ring-primary"
        >
          <Search class="h-4 w-4 text-muted-foreground mr-2" />
          <input
            type="text"
            placeholder="Search applications..."
            class="flex-1 bg-transparent outline-none text-sm"
            bind:value={searchTerm}
          />
        </div>
      </div>
    </div>
  </div>

  {#if error}
    <div class="rounded-xl border bg-destructive/5 p-6 animate-fade-in">
      <div class="flex items-center gap-3">
        <div class="p-2 bg-destructive/10 rounded-full">
          <Zap class="h-5 w-5 text-destructive" />
        </div>
        <div>
          <h3 class="text-lg font-medium text-destructive">
            Error loading applications
          </h3>
          <p class="text-sm text-muted-foreground mt-1">{error}</p>
        </div>
      </div>
      <div class="mt-4 flex justify-end">
        <button
          class="inline-flex items-center justify-center rounded-md bg-primary px-4 py-2 text-sm font-medium text-primary-foreground hover:bg-primary/90 transition-colors"
          onclick={() => window.location.reload()}
        >
          Try Again
        </button>
      </div>
    </div>
  {:else}
    

    <div class="flex flex-col md:flex-row gap-6">
        <!-- Main Content - Applications Grid -->
      <div class="w-full md:w-4/4 flex flex-col h-[calc(100vh-16rem)]">
        <!-- Adjust -16rem as needed based on header size -->
        <div
          class="rounded-xl border bg-card shadow-sm flex-grow flex flex-col overflow-hidden h-full animate-fade-in"
          style="animation-delay: 150ms;"
        >
          <div class="p-5 border-b">
            <div class="flex items-center justify-between">
              <h3 class="text-lg font-medium flex items-center gap-2">
                <Zap class="h-5 w-5 text-primary" />
                Applications
              </h3>
              <div class="flex items-center gap-2">
                <Badge variant="outline" class="bg-muted/30">
                  {accessibleClients.length} apps
                </Badge>
              </div>
            </div>
          </div>

          <div class="flex-grow overflow-auto p-5 pt-3">
            {#if accessibleClients.length === 0}
              <div
                class="flex flex-col items-center justify-center py-12 text-center bg-muted/5 rounded-lg border-dashed border-2 my-4 mx-2"
              >
                <div class="rounded-full bg-muted/30 p-4 mb-4">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    class="text-muted-foreground"
                  >
                    <rect width="8" height="8" x="2" y="2" rx="1"></rect>
                    <path d="M6 6h.01"></path>
                    <rect width="8" height="8" x="14" y="2" rx="1"></rect>
                    <path d="M18 6h.01"></path>
                    <rect width="8" height="8" x="2" y="14" rx="1"></rect>
                    <path d="M6 18h.01"></path>
                    <rect width="8" height="8" x="14" y="14" rx="1"></rect>
                    <path d="M18 18h.01"></path>
                  </svg>
                </div>
                <h3 class="text-lg font-medium">No applications found</h3>
                <p class="text-sm text-muted-foreground mt-1 max-w-sm mx-auto">
                  You don't have access to any applications yet. Contact your
                  administrator to request access.
                </p>
              </div>
            {:else}
              <div class="h-full relative">
                    <div
                      class="grid gap-4 grid-cols-1 sm:grid-cols-2 md:grid-cols-3 w-full h-full"
                    >
                      {#each data.clients.data as client, i}
                        <div
                          class="min-h-[200px] animate-fade-in opacity-0"
                          style="animation-delay: {200 + i * 75}ms"
                        >
                          <ClientCard
                            {client}
                            index={i}
                            columns={gridColumns}
                          />
                        </div>
                      {/each}
                    </div>
              </div>
            {/if}
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>
