<script>
  import { v1 as uuid } from "uuid";
  import axios from "axios";

  const algorithmList = ["Boyer-Moore", "KMP", "Regex"];

  let files = [];
  let keyword = "";
  let algorithm = 0;
  let resultPromises = [];
  let server = "http://localhost:3000/";
  let submittedKeywordLength = 0;

  function addFiles(list) {
    for (let i = 0; i < list.length; ++i) {
      const reader = new FileReader();
      reader.readAsText(list[i], "UTF-8");
      reader.onload = e => {
        files = [
          ...files,
          {
            id: uuid(),
            filename: list[i].name,
            text: e.target.result
          }
        ];
      };
    }
  }

  function removeFile(id) {
    files = files.filter(f => f.id !== id);
  }

  function search() {
    resultPromises = [];
    submittedKeywordLength = keyword.length;
    for (let i = 0; i < files.length; ++i) {
      resultPromises.push({
        id: files[i].id,
        filename: files[i].filename,
        promise: axios({
          method: "POST",
          baseURL: server,
          data: {
            text: files[i].text,
            keyword: keyword,
            algorithm: algorithmList[algorithm]
          }
        })
      });
    }
  }
</script>

<label class="btn btn-primary">
  <input
    type="file"
    style="display: none;"
    multiple
    on:change={e => {
      addFiles(Array.from(e.target.files));
      e.target.value = null;
    }} />
  <span>Tambahkan File</span>
</label>
<div class="accordion">
  {#each files as file (file.id)}
    <div class="card">
      <div class="card-header" id="card-heading">
        <h2 class="mb-0" style="display: flex; justify-content: space-between;">
          <button
            class="btn btn-link"
            type="button"
            data-toggle="collapse"
            data-target={`#collapse-${file.id}`}
            aria-expanded="false"
            aria-controls={'collapse-' + file.id}>
            {file.filename}
          </button>
          <button class="btn btn-danger" on:click={() => removeFile(file.id)}>
            Hapus
          </button>
        </h2>
      </div>
      <div
        id={'collapse-' + file.id}
        class="collapse"
        aria-labelledby="card-heading">
        <pre class="card-body text-justify">{file.text}</pre>
      </div>
    </div>
  {:else}
    <p>Belum ada file yang dipilih...</p>
  {/each}
</div>
<div class="mt-4">
  <form>
    <div class="form-group">
      <label for="keyword">
        <b>Keyword:</b>
      </label>
      <input
        type="text"
        class="form-control"
        id="keyword"
        bind:value={keyword} />
    </div>
    <div class="form-group">
      <b>Pilih algoritma:</b>
      <div class="form-check">
        <input
          class="form-check-input"
          type="radio"
          bind:group={algorithm}
          id="option0"
          value={0} />
        <label for="option0" class="form-check-label">Boyer-Moore</label>
      </div>
      <div class="form-check">
        <input
          class="form-check-input"
          type="radio"
          bind:group={algorithm}
          id="option1"
          value={1} />
        <label for="option1" class="form-check-label">Knuth-Morris-Pratt</label>
      </div>
      <div class="form-check">
        <input
          class="form-check-input"
          type="radio"
          bind:group={algorithm}
          id="option2"
          value={2} />
        <label for="option2" class="form-check-label">Regular Expression</label>
      </div>
    </div>
    <form class="form-inline">
      <button
        class="btn btn-primary mr-2"
        on:click|preventDefault={search}
        disabled={!server || !keyword || !files.length}>
        Cari
      </button>
      <input
        type="text"
        class="form-control"
        bind:value={server}
        placeholder="API URL" />
    </form>
  </form>
</div>
{#if resultPromises.length > 0}
  <div class="accordion mt-4">
    <hr />
    <h4>Hasil:</h4>
    {#each resultPromises as resultPromise}
      <div class="card">
        <div class="card-header" id="card-result-heading">
          <h2
            class="mb-0"
            style="display: flex; justify-content: space-between;">
            <button
              class="btn btn-link"
              type="button"
              data-toggle="collapse"
              data-target={`#collapse-result-${resultPromise.id}`}
              aria-expanded="true"
              aria-controls={'collapse-result-' + resultPromise.id}>
              {resultPromise.filename}
            </button>
          </h2>
        </div>
        <div
          id={'collapse-result-' + resultPromise.id}
          class="collapse show"
          aria-labelledby="card-result-heading">
          {#await resultPromise.promise}
            Fetching jawaban...
          {:then result}
            {#if result.data && result.data.length}
              {#each result.data as data}
                <div class="card">
                  <div class="card-body">
                    <b>Waktu:</b>
                    {data.time}
                    <br />
                    <b>Jumlah:</b>
                    {data.count}
                    <br />
                    <b>Kalimat:</b>
                    <br />
                    {@html data.sentence.substring(0, data.index_found) + data.sentence
                        .substring(
                          data.index_found,
                          data.index_found + submittedKeywordLength
                        )
                        .bold() + data.sentence.substring(data.index_found + submittedKeywordLength)}
                  </div>
                </div>
              {/each}
            {:else}
              <div class="card">
                <div class="card-body">Pencarian tidak ditemukan.</div>
              </div>
            {/if}
          {:catch error}
            <pre class="text-danger">
              {@html error.response ? (error.response.data ? error.response.data : `Error ${error.response.status}`) : 'Unknown error occured!'}
            </pre>
          {/await}
        </div>
      </div>
    {/each}
  </div>
{/if}
