<script lang="ts">
  import { Input } from "$lib/components/ui/input/index.js";
  import { Label } from "$lib/components/ui/label/index.js";
  import { Textarea } from "$lib/components/ui/textarea/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import { Switch } from "$lib/components/ui/switch/index.js";
  import { Badge } from "$lib/components/ui/badge/index.js";
  import {
    Card,
    CardContent,
    CardHeader,
    CardTitle,
  } from "$lib/components/ui/card/index.js";
  import { Separator } from "$lib/components/ui/separator/index.js";
  import { Alert, AlertDescription } from "$lib/components/ui/alert/index.js";
  import { Trash2, Plus, Download, AlertCircle } from "@lucide/svelte";

  // Form state
  let formData = $state({
    // Required properties
    name: "",
    version: "",

    // Recommended properties
    description: "",
    displayName: "",
    unity: "",

    // Optional properties
    author: {
      name: "",
      email: "",
      url: "",
    },
    changelogUrl: "",
    dependencies: [] as Array<{ name: string; version: string }>,
    documentationUrl: "",
    hideInEditor: false,
    keywords: [] as string[],
    license: "",
    licensesUrl: "",
    samples: [] as Array<{
      displayName: string;
      description: string;
      path: string;
    }>,
    type: "",
    unityRelease: "",
  });

  let newKeyword = $state("");
  let newDependency = $state({ name: "", version: "" });
  let newSample = $state({ displayName: "", description: "", path: "" });

  // Validation state
  let errors = $state({
    name: "",
    version: "",
    unity: "",
    unityRelease: "",
    dependencies: [] as { name: string; version: string }[],
    author: { email: "", url: "" },
  });

  let isFormValid = $derived(
    !errors.name &&
      !errors.version &&
      !errors.unity &&
      !errors.unityRelease &&
      !errors.author.email &&
      !errors.author.url &&
      errors.dependencies.every((dep) => !dep.name && !dep.version) &&
      formData.name.trim() != "" &&
      formData.version.trim() != ""
  );

  // Validation functions
  function validatePackageName(name: string): string {
    if (!name.trim()) return "Package name is required";
    const namePattern = /^[a-z0-9-]+(\.[a-z0-9-]+)+$/;
    if (!namePattern.test(name)) {
      return "Must follow reverse domain notation (e.g., com.company.package-name)";
    }
    return "";
  }

  function validateVersion(version: string): string {
    if (!version.trim()) return "Version is required";
    const versionPattern = /^\d+\.\d+\.\d+$/;
    if (!versionPattern.test(version)) {
      return "Must follow semantic versioning (e.g., 1.0.0)";
    }
    return "";
  }

  function validateUnityVersion(unity: string): string {
    if (!unity) return "";
    const unityPattern = /^\d{4}\.\d+$/;
    if (!unityPattern.test(unity)) {
      return "Must follow Unity version format (e.g., 2019.1)";
    }
    return "";
  }

  function validateUnityRelease(release: string): string {
    if (!release) return "";
    const releasePattern = /^\d+[ab]\d+$/;
    if (!releasePattern.test(release)) {
      return "Must follow Unity release format (e.g., 0b4)";
    }
    return "";
  }

  function validateEmail(email: string): string {
    if (!email) return "";
    const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailPattern.test(email)) {
      return "Invalid email format";
    }
    return "";
  }

  function validateUrl(url: string): string {
    if (!url) return "";
    try {
      new URL(url);
      return "";
    } catch {
      return "Invalid URL format";
    }
  }

  function validateDependencyName(name: string): string {
    if (!name.trim()) return "Package name is required";
    return validatePackageName(name);
  }

  function validateDependencyVersion(version: string): string {
    if (!version.trim()) return "Version is required";
    return validateVersion(version);
  }

  // Reactive validation
  $effect(() => {
    errors.name = validatePackageName(formData.name);
  });

  $effect(() => {
    errors.version = validateVersion(formData.version);
  });

  $effect(() => {
    errors.unity = validateUnityVersion(formData.unity);
  });

  $effect(() => {
    errors.unityRelease = validateUnityRelease(formData.unityRelease);
  });

  $effect(() => {
    errors.author.email = validateEmail(formData.author.email);
  });

  $effect(() => {
    errors.author.url = validateUrl(formData.author.url);
  });

  function addKeyword() {
    if (newKeyword.trim() && !formData.keywords.includes(newKeyword.trim())) {
      formData.keywords.push(newKeyword.trim());
      newKeyword = "";
    }
  }

  function removeKeyword(index: number) {
    formData.keywords.splice(index, 1);
  }

  function addDependency() {
    const nameError = validateDependencyName(newDependency.name);
    const versionError = validateDependencyVersion(newDependency.version);

    errors.dependencies.push({ name: nameError, version: versionError });

    if (!nameError && !versionError) {
      formData.dependencies.push({ ...newDependency });
      newDependency = { name: "", version: "" };
    }
  }

  function updateDependencyName(
    evt: Event & {
      currentTarget: EventTarget & HTMLInputElement;
    },
    index: number
  ) {
    formData.dependencies[index].name = evt.currentTarget.value;
    errors.dependencies[index].name = validateDependencyName(
      evt.currentTarget.value
    );
  }

  function updateDependencyVersion(
    evt: Event & {
      currentTarget: EventTarget & HTMLInputElement;
    },
    index: number
  ) {
    formData.dependencies[index].version = evt.currentTarget.value;
    errors.dependencies[index].version = validateDependencyVersion(
      evt.currentTarget.value
    );
  }

  function removeDependency(index: number) {
    formData.dependencies.splice(index, 1);
    errors.dependencies.splice(index, 1);
  }

  function addSample() {
    if (newSample.displayName.trim() && newSample.path.trim()) {
      formData.samples.push({ ...newSample });
      newSample = { displayName: "", description: "", path: "" };
    }
  }

  function removeSample(index: number) {
    formData.samples.splice(index, 1);
  }

  function handleKeywordKeypress(event: KeyboardEvent) {
    if (event.key === "Enter") {
      event.preventDefault();
      addKeyword();
    }
  }

  // File generation and download
  function generatePackageJson() {
    const manifest: any = {
      name: formData.name,
      version: formData.version,
    };

    // Add recommended properties
    if (formData.displayName) manifest.displayName = formData.displayName;
    if (formData.description) manifest.description = formData.description;
    if (formData.unity) manifest.unity = formData.unity;

    // Add optional properties
    if (formData.author.name || formData.author.email || formData.author.url) {
      manifest.author = {};
      if (formData.author.name) manifest.author.name = formData.author.name;
      if (formData.author.email) manifest.author.email = formData.author.email;
      if (formData.author.url) manifest.author.url = formData.author.url;
    }

    if (formData.changelogUrl) manifest.changelogUrl = formData.changelogUrl;
    if (formData.documentationUrl)
      manifest.documentationUrl = formData.documentationUrl;
    if (formData.licensesUrl) manifest.licensesUrl = formData.licensesUrl;
    if (formData.license) manifest.license = formData.license;
    if (formData.unityRelease) manifest.unityRelease = formData.unityRelease;
    if (formData.hideInEditor) manifest.hideInEditor = formData.hideInEditor;
    if (formData.type) manifest.type = formData.type;

    if (formData.keywords.length > 0) manifest.keywords = formData.keywords;

    if (formData.dependencies.length > 0) {
      manifest.dependencies = {};
      formData.dependencies.forEach((dep) => {
        manifest.dependencies[dep.name] = dep.version;
      });
    }

    if (formData.samples.length > 0) {
      manifest.samples = formData.samples.map((sample) => ({
        displayName: sample.displayName,
        description: sample.description,
        path: sample.path,
      }));
    }

    return manifest;
  }

  function downloadPackageJson() {
    if (!isFormValid) return;

    const manifest = generatePackageJson();
    const jsonString = JSON.stringify(manifest, null, 2);
    const blob = new Blob([jsonString], { type: "application/json" });
    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = url;
    a.download = "package.json";
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function handleSubmit(event: Event) {
    event.preventDefault();
    if (isFormValid) {
      downloadPackageJson();
    }
  }
</script>

<form
  class="not-3xl:space-y-8 3xl:grid 3xl:grid-cols-2 3xl:gap-4"
  onsubmit={handleSubmit}
>
  <!-- Required Properties -->
  <Card>
    <CardHeader>
      <CardTitle class="text-xl font-semibold">Required Properties</CardTitle>
    </CardHeader>
    <CardContent class="space-y-4">
      <div class="grid-2-cols">
        <div class="field-container">
          <Label for="name">Package Name *</Label>
          <Input
            id="name"
            bind:value={formData.name}
            placeholder="com.company.package-name"
            required
            class={errors.name ? "border-destructive" : ""}
          />
          {#if errors.name}
            <p class="text-sm text-destructive">{errors.name}</p>
          {:else}
            <p class="help-text">
              Must follow reverse domain name notation (e.g.,
              com.unity.timeline)
            </p>
          {/if}
        </div>

        <div class="field-container">
          <Label for="version">Version *</Label>
          <Input
            id="version"
            bind:value={formData.version}
            placeholder="1.0.0"
            pattern="^\d+\.\d+\.\d+$"
            required
            class={errors.version ? "border-destructive" : ""}
          />
          {#if errors.version}
            <p class="text-sm text-destructive">{errors.version}</p>
          {:else}
            <p class="help-text">
              Semantic versioning format: major.minor.patch
            </p>
          {/if}
        </div>
      </div>
    </CardContent>
  </Card>

  <!-- Recommended Properties -->
  <Card>
    <CardHeader>
      <CardTitle class="text-xl font-semibold">Recommended Properties</CardTitle
      >
    </CardHeader>
    <CardContent class="space-y-4">
      <div class="field-container">
        <Label for="displayName">Display Name</Label>
        <Input
          id="displayName"
          bind:value={formData.displayName}
          placeholder="My Awesome Package"
        />
        <p class="help-text">User-friendly name that appears in Unity Editor</p>
      </div>

      <div class="field-container">
        <Label for="description">Description</Label>
        <Textarea
          id="description"
          bind:value={formData.description}
          placeholder="A brief description of what this package does..."
          rows={3}
        />
        <p class="help-text">
          Supports UTF-8 characters including line breaks (\n) and bullets
          (\u25AA)
        </p>
      </div>

      <div class="field-container">
        <Label for="unity">Unity Version</Label>
        <Input
          id="unity"
          bind:value={formData.unity}
          placeholder="2019.1"
          pattern="^\d{4}\.\d+$"
          class={errors.unity ? "border-destructive" : ""}
        />
        {#if errors.unity}
          <p class="text-sm text-destructive">{errors.unity}</p>
        {:else}
          <p class="help-text">
            Lowest compatible Unity version (format: major.minor, e.g.,
            "2019.1")
          </p>
        {/if}
      </div>
    </CardContent>
  </Card>

  <!-- Optional Properties -->
  <Card>
    <CardHeader>
      <CardTitle class="text-xl font-semibold">Optional Properties</CardTitle>
    </CardHeader>
    <CardContent class="space-y-6">
      <!-- Author Information -->
      <div class="space-y-4">
        <h4 class="section-title">Author Information</h4>
        <div class="grid-3-cols">
          <div class="field-container">
            <Label for="authorName">Author Name</Label>
            <Input
              id="authorName"
              bind:value={formData.author.name}
              placeholder="John Doe"
            />
          </div>
          <div class="field-container">
            <Label for="authorEmail">Author Email</Label>
            <Input
              id="authorEmail"
              type="email"
              bind:value={formData.author.email}
              placeholder="john.doe@example.com"
              class={errors.author.email ? "border-destructive" : ""}
            />
            {#if errors.author.email}
              <p class="text-sm text-destructive">{errors.author.email}</p>
            {/if}
          </div>
          <div class="field-container">
            <Label for="authorUrl">Author URL</Label>
            <Input
              id="authorUrl"
              type="url"
              bind:value={formData.author.url}
              placeholder="https://johndoe.com"
              class={errors.author.url ? "border-destructive" : ""}
            />
            {#if errors.author.url}
              <p class="text-sm text-destructive">{errors.author.url}</p>
            {/if}
          </div>
        </div>
      </div>

      <!-- URLs -->
      <div class="space-y-4">
        <h4 class="section-title">Documentation & Links</h4>
        <div class="grid-3-cols">
          <div class="field-container">
            <Label for="documentationUrl">Documentation URL</Label>
            <Input
              id="documentationUrl"
              type="url"
              bind:value={formData.documentationUrl}
              placeholder="https://example.com/docs"
            />
          </div>
          <div class="field-container">
            <Label for="changelogUrl">Changelog URL</Label>
            <Input
              id="changelogUrl"
              type="url"
              bind:value={formData.changelogUrl}
              placeholder="https://example.com/changelog"
            />
          </div>
          <div class="field-container">
            <Label for="licensesUrl">License URL</Label>
            <Input
              id="licensesUrl"
              type="url"
              bind:value={formData.licensesUrl}
              placeholder="https://example.com/license"
            />
          </div>
        </div>
      </div>

      <!-- License and Unity Release -->
      <div class="grid-2-cols">
        <div class="field-container">
          <Label for="license">License</Label>
          <Input
            id="license"
            bind:value={formData.license}
            placeholder="MIT or Refer to LICENSE.md file"
          />
          <p class="help-text">SPDX identifier or custom license text</p>
        </div>
        <div class="field-container">
          <Label for="unityRelease">Unity Release</Label>
          <Input
            id="unityRelease"
            bind:value={formData.unityRelease}
            placeholder="0b4"
            pattern="^\d+[ab]\d+$"
            class={errors.unityRelease ? "border-destructive" : ""}
          />
          <p class="help-text">Specific Unity release (e.g., "0b4" for beta)</p>
          {#if errors.unityRelease}
            <p class="text-sm text-destructive">{errors.unityRelease}</p>
          {/if}
        </div>
      </div>

      <Separator />

      <!-- Keywords -->
      <div class="space-y-4">
        <h4 class="section-title">Keywords</h4>
        <div class="keyword-container">
          {#each formData.keywords as keyword, index}
            <button
              type="button"
              onclick={() => removeKeyword(index)}
              class="remove-button"
            >
              <Badge variant="outline" class="keyword-badge">
                {keyword}
                <Trash2 size={12} />
              </Badge>
            </button>
          {/each}
        </div>
        <div class="add-item-row">
          <Input
            bind:value={newKeyword}
            placeholder="Add keyword..."
            onkeypress={handleKeywordKeypress}
          />
          <Button type="button" onclick={addKeyword} variant="outline">
            <Plus class="icon-button" />
          </Button>
        </div>
      </div>

      <!-- Dependencies -->
      <div class="space-y-4">
        <h4 class="section-title">Dependencies</h4>
        {#each formData.dependencies as dependency, index}
          <div class="dynamic-item">
            <Input
              value={dependency.name}
              onchange={(evt) => updateDependencyName(evt, index)}
              class={errors.dependencies[index].name
                ? "border-destructive"
                : ""}
            />
            <Input
              value={dependency.version}
              onchange={(evt) => updateDependencyVersion(evt, index)}
              class={errors.dependencies[index].version
                ? "border-destructive"
                : ""}
            />
            <Button
              type="button"
              onclick={() => removeDependency(index)}
              variant="outline"
              size="sm"
            >
              <Trash2 class="icon-button" />
            </Button>
          </div>
        {/each}
        <div class="add-item-row">
          <Input
            bind:value={newDependency.name}
            placeholder="Package name (e.g., com.unity.timeline)"
          />
          <Input
            bind:value={newDependency.version}
            placeholder="Version (e.g., 1.0.0)"
          />
          <Button type="button" onclick={addDependency} variant="outline">
            <Plus class="icon-button" />
          </Button>
        </div>
      </div>

      <!-- Samples -->
      <div class="space-y-4">
        <h4 class="section-title">Samples</h4>
        {#each formData.samples as sample, index}
          <div class="dynamic-item dynamic-sample-item">
            <div class="sample-header">
              <h5 class="font-medium">{sample.displayName}</h5>
              <Button
                type="button"
                onclick={() => removeSample(index)}
                variant="outline"
                size="sm"
              >
                <Trash2 class="icon-button" />
              </Button>
            </div>
            <p class="help-text">{sample.description}</p>
            <p class="sample-path">{sample.path}</p>
          </div>
        {/each}
        <div class="grid-2-cols">
          <Input
            bind:value={newSample.displayName}
            placeholder="Sample display name"
          />
          <div class="add-item-row">
            <Input
              bind:value={newSample.path}
              placeholder="Samples~/SampleFolder"
            />
            <Button type="button" onclick={addSample} variant="outline">
              <Plus class="icon-button" />
            </Button>
          </div>
          <Textarea
            bind:value={newSample.description}
            placeholder="Sample description"
            class="col-span-2"
          />
        </div>
      </div>
    </CardContent>
  </Card>

  <div class="not-3xl:space-y-8 space-y-4">
    <!-- Advanced Options -->
    <Card>
      <CardHeader>
        <CardTitle class="text-xl font-semibold">Advanced Options</CardTitle>
      </CardHeader>
      <CardContent class="space-y-6">
        <div class="space-y-4">
          <div class="grid-2-cols">
            <div class="field-container">
              <Label for="type">Package Type</Label>
              <Input
                id="type"
                bind:value={formData.type}
                placeholder="Reserved for internal use"
                readonly
              />
              <p class="help-text">Reserved for internal Unity use</p>
            </div>
            <div class="field-container">
              <Label for="hideInEditor">Hide in Editor</Label>
              <div class="flex items-center h-8">
                <Switch
                  id="hideInEditor"
                  bind:checked={formData.hideInEditor}
                />
              </div>
              <p class="help-text">
                Hide package assets in Project window and Object Picker
              </p>
            </div>
          </div>
        </div>
      </CardContent>
    </Card>

    {#if isFormValid}
      <Button type="submit" class="w-full" disabled={!isFormValid}>
        <Download></Download>
        Download
      </Button>
    {:else}
      <Alert variant="destructive">
        <AlertCircle class="h-4 w-4" />
        <AlertDescription>
          Please fix the issues before generating the package manifest.
        </AlertDescription>
      </Alert>
    {/if}
  </div>
</form>

<style lang="postcss">
  @reference "tailwindcss/theme";
  .field-container {
    @apply space-y-2;
  }

  .remove-button {
    @apply hover:opacity-50 hover:cursor-pointer;
  }

  .grid-2-cols {
    @apply grid grid-cols-1 md:grid-cols-2 gap-4;
  }

  .grid-3-cols {
    @apply grid grid-cols-1 md:grid-cols-3 gap-4;
  }

  .help-text {
    @apply text-sm;
  }

  .sample-path {
    @apply text-sm text-gray-600 dark:text-gray-400;
  }

  .dynamic-item {
    @apply flex items-center gap-2;
  }

  .section-title {
    @apply text-lg font-medium;
  }

  .add-item-row {
    @apply flex items-center gap-2;
  }

  .dynamic-sample-item {
    @apply border rounded-lg p-3 flex flex-col items-stretch;
  }

  .sample-header {
    @apply flex items-stretch justify-between;
  }
</style>
