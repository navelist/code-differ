<script lang='ts'>
 
    import { page } from '$app/stores';
    
    // e.g. https://raw.githubusercontent.com/navelist/freethinkers/5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c/README.md
    // encoded https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c%2FREADME.md
    let urlFrom: string = "https://raw.githubusercontent.com/navelist/freethinkers/5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c/README.md";// = $page.url.searchParams.get('from');
    // e.g. https://raw.githubusercontent.com/navelist/freethinkers/7b83b462c89a2457798195a0278c30d5bbbd6592/README.md
    // encoded https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F7b83b462c89a2457798195a0278c30d5bbbd6592%2FREADME.md
    let urlTo: string = "https://raw.githubusercontent.com/navelist/freethinkers/7b83b462c89a2457798195a0278c30d5bbbd6592/README.md";// = $page.url.searchParams.get('to');
    // example url : http://localhost:5173/?from=https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c%2FREADME.md&to=https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F7b83b462c89a2457798195a0278c30d5bbbd6592%2FREADME.md
    
    let textFrom: string = "";
    let textTo: string = "";

    function fetchFiles() {
        fetch(urlFrom).then(response => response.text()).then(response => textFrom = response );
        fetch(urlTo).then(response => response.text()).then(response => textTo = response );
    }

    $: if (textFrom && textTo) {
        // console.log(textFrom);
        // console.log(textTo);
    }



</script>

<div>
    <div>
        <div>
            Compare From : <input value={urlFrom}>
        </div>
        <div>
            Compare To : <input value={urlTo}>
        </div>
        <div>
            <button on:click={fetchFiles}> Start </button>
        </div>
    </div>
    <div>
        <table>
            <thead hidden>
                <tr>
                    <th scope=col> original file line </th>
                    <th scope=col> original file line number </th>
                    <th scope=col> diff file line number </th>
                    <th scope=col> diff file line </th>
                </tr>
            </thead>
            <colgroup>
                <col>
                <col width="44">
                <col width="44">
                <col>
            </colgroup>
            <tbody>
                <tr>
                    <td class="original_line"> line number </td>
                    <td class="original_line_number"> original line </td>
                    <td class="diff_line_number"> line number </td>
                    <td class="diff_line"> diff line </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

<style>
    :global(html) {
        min-height:100%;
    }

    table {
        min-width:100%;
        table-layout:fixed;
    }
</style>