<script>
  import Dropzone from "svelte-file-dropzone";

  let files = {
    accepted: [],
  };

  let soundJson = {};
  let minify = true;

  $: theme = localStorage.getItem(theme);

  function handleFilesSelect(e) {
    const { acceptedFiles } = e.detail;
    files.accepted = [...files.accepted, ...acceptedFiles];

    files.accepted.forEach((el) => {
      const id = el.path
        .replaceAll("/", ".")
        .replace(".ogg", "")
        .replace(".sounds.", "");
      const name = el.path.replace(".ogg", "").replace("/sounds/", "");
      const json = {};
      json[id] = { sounds: [{ name, stream: true }] };
      soundJson = { ...soundJson, ...json };
    });
  }

  function downloadJson() {
    let json;
    if (minify) {
      json = JSON.stringify(soundJson);
    } else {
      json = JSON.stringify(soundJson, null, 2);
    }

    const blob = new Blob([json], { type: "application/json" });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "sounds.json";
    a.click();
    a.remove();
    window.URL.revokeObjectURL(url);
  }

  function clear() {
    files.accepted = [];
    soundJson = {};
  }
</script>

<div
  class="min-w-screen min-h-screen flex items-center justify-center px-5 py-5 bg-base-300"
>
  <div
    class="w-full mx-auto rounded-lg shadow p-5 bg-base-100"
    style="max-width: 800px"
  >
    <Dropzone
      on:drop={handleFilesSelect}
      accept=".ogg"
      noClick
      disableDefaultStyles="true"
      containerClasses="flex justify-center my-4 h-32 border-2 border-gray-300 border-dashed rounded-md hover:border-gray-400"
    >
      <span class="flex items-center space-x-2">
        <span class="font-medium text-gray-600"> 여기에 sounds폴더 드롭 </span>
      </span>
    </Dropzone>

    <div class="flex -mx-2 mb-2">
      <div class="w-1/2 px-2">
        <input
          type="checkbox"
          class="align-middle checkbox"
          bind:checked={minify}
        />
        <span class="text-xs font-bold text-gray-500">JSON 압축</span>
      </div>
    </div>
    <div class="sm:space-x-4 space-y-2 py-4 text-center ">
      <button class="btn btn-secondary btn-wide" on:click={clear}>초기화</button
      >
      <button
        class="btn btn-primary btn-wide"
        disabled={JSON.stringify(soundJson) === "{}"}
        on:click={downloadJson}>다운로드</button
      >
    </div>
    {#if files.accepted.length > 0}
      <div class="collapse collapse-arrow">
        <input type="checkbox" />
        <div class="collapse-title text-xl font-medium">파일 목록</div>
        <div class="collapse-content">
          <ul>
            {#each files.accepted as item}
              <li>{item.path}</li>
            {/each}
          </ul>
        </div>
      </div>
    {/if}
  </div>
</div>
