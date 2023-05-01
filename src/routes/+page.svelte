<script lang='ts'>
    import { type Change, diffWords } from "diff"

    interface ChangeObject{
        idiff: number, // > 0 : addition, < 0 : removal, == 0 : plain 
        value: string,
        newline?: boolean
    }

    type ChangeObjectArray = Array<ChangeObject>;
    type ChangeObjectLine = Array<ChangeObject> | null;
    type ChangeObjectLines = Array<ChangeObjectLine>;
    interface PrintableLine {
        aLine: ChangeObjectLine,
        aIndex: number,
        bLine: ChangeObjectLine,
        bIndex: number
    }
    type PrintableChanges = Array<PrintableLine>;
    const nonRenderedVariables = {
        changeLinesLeft : [] as ChangeObjectLines,
        changeLinesRight : [] as ChangeObjectLines,

        // https://github.com/navelist/freethinkers/commit/b84be598021d580d5396601644c076f366a82705
        // urlFrom : "https://raw.githubusercontent.com/navelist/freethinkers/fa29110d6b7b365cc28a324c2458adf43f0ccf08/README.md",
        // urlTo : "https://raw.githubusercontent.com/navelist/freethinkers/b84be598021d580d5396601644c076f366a82705/README.md",

        // https://github.com/navelist/freethinkers/compare/fa29110d6b7b365cc28a324c2458adf43f0ccf08..b84be598021d580d5396601644c076f366a82705
        // e.g. https://raw.githubusercontent.com/navelist/freethinkers/5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c/README.md
        // encoded https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c%2FREADME.md
        urlFrom : "https://raw.githubusercontent.com/navelist/freethinkers/fa29110d6b7b365cc28a324c2458adf43f0ccf08/README.md",// = $page.url.searchParams.get('from');
        // e.g. https://raw.githubusercontent.com/navelist/freethinkers/7b83b462c89a2457798195a0278c30d5bbbd6592/README.md
        // encoded https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F7b83b462c89a2457798195a0278c30d5bbbd6592%2FREADME.md
        urlTo : "https://raw.githubusercontent.com/navelist/freethinkers/b84be598021d580d5396601644c076f366a82705/README.md",// = $page.url.searchParams.get('to');
        // example url : http://localhost:5173/?from=https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F5f23f7c6462ed6bacd5208448bf0b2ce9b372c0c%2FREADME.md&to=https%3A%2F%2Fraw.githubusercontent.com%2Fnavelist%2Ffreethinkers%2F7b83b462c89a2457798195a0278c30d5bbbd6592%2FREADME.md
        textFrom : "",
        textTo : "",
    };

    let printableChanges: PrintableChanges = [];
    
    

    function fetchFiles() {
        const promiseA = fetch(nonRenderedVariables.urlFrom)
            .then(response => response.text())
            .then(response => response.replaceAll("\r\n", "\n"))
            .then(response => nonRenderedVariables.textFrom = response );
        const promiseB = fetch(nonRenderedVariables.urlTo)
            .then(response => response.text())
            .then(response => response.replaceAll("\r\n", "\n"))
            .then(response => nonRenderedVariables.textTo = response );

        Promise.all([promiseA, promiseB]).then(_ => createChangeObjects());
    }

    function splitLineAndCreateChangeObjects(co: Change) {
        const lines = co.value.split("\n").map(line=> line+"\n")
        
        // The last line should not have new line feed
        for (let i = lines.length -1 ; i >= 0; i--) {
            if(lines[i]!="\n") {
                lines[i]=lines[i].replace("\n", "");
                break;
            }
        }
        
        const coLines = lines.map(line=> {
            return {
                idiff : co.added ? 1 : co.removed ? -1 : 0,
                value : line.replace("\n", ""),
                newline : line.includes("\n")
            }
        })
        // if (co.added || co.removed) {
        //     console.debug(co);
        //     console.debug(coLines);
        // }
        return coLines;
    }

    function createLeftRightLines(changeObjectArray : ChangeObjectArray) {
        let leftLines : ChangeObjectLines = [];
        let rightLines : ChangeObjectLines = [];

        let leftLine : ChangeObjectArray = [];
        let rightLine : ChangeObjectArray = [];

        function flushLeft() {
            leftLines.push(leftLine);
            leftLine = [];
        }
        function flushRight() {
            rightLines.push(rightLine);
            rightLine = [];
        }
        function flushBoth() {
            flushLeft();
            flushRight();
        }

        for (const co of changeObjectArray) {
            if (co.idiff == 0) {
                leftLine.push(co);
                rightLine.push(co);
                if (co.newline) {
                    flushBoth();
                }
            }
            else if (co.idiff < 0) {
                leftLine.push(co);
                if (co.newline) {
                    flushLeft();
                }
            }else if (co.idiff > 0) {
                rightLine.push(co);
                if (co.newline) {
                    flushRight();
                }
            }
        }

        nonRenderedVariables.changeLinesLeft = leftLines;
        nonRenderedVariables.changeLinesRight = rightLines;
    }

    function generatePrintableChanges() {
        const _printableChanges : PrintableChanges = [];
        let leftIndex = 1;
        let rightIndex = 1;

        for (let i = 0; i < Math.max(nonRenderedVariables.changeLinesLeft.length, nonRenderedVariables.changeLinesRight.length); i++) {
            const _printableLine : PrintableLine = {
                aIndex : 0,
                bIndex : 0,
                aLine : null,
                bLine : null,
            };
            if (i < nonRenderedVariables.changeLinesLeft.length) {
                const left = nonRenderedVariables.changeLinesLeft[i];
                if (left) {
                    _printableLine.aLine = left;
                    _printableLine.aIndex = leftIndex ++;
                }
            }
            if (i < nonRenderedVariables.changeLinesRight.length) {
                const right = nonRenderedVariables.changeLinesRight[i];
                if (right) {
                    _printableLine.bLine = right;
                    _printableLine.bIndex = rightIndex ++;
                }
            }
            _printableChanges.push(_printableLine);
        }
        printableChanges = _printableChanges;
    }

    function createChangeObjects() {
        const diffResult = diffWords(nonRenderedVariables.textFrom, nonRenderedVariables.textTo);
        let changeObjectArray : ChangeObjectArray = [];
        for (const co of diffResult) {
            changeObjectArray = changeObjectArray.concat(splitLineAndCreateChangeObjects(co));
        }
        console.debug(changeObjectArray);
        createLeftRightLines(changeObjectArray);
        
        console.debug(nonRenderedVariables.changeLinesLeft);
        console.debug(nonRenderedVariables.changeLinesRight);
        generatePrintableChanges();
    }



</script>

<div>
    <div>
        <div>
            Compare From : <input value={nonRenderedVariables.urlFrom}>
        </div>
        <div>
            Compare To : <input value={nonRenderedVariables.urlTo}>
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
                {#each printableChanges as printableLine}
                <tr>
                    <td>
                        {#if printableLine.aLine}
                            {#each printableLine.aLine as co}
                                <span class={"diff"+co.idiff}> {co.value} </span>
                            {/each}
                        {:else}
                            <!-- empty -->
                        {/if}
                    </td>
                    <td>
                        {#if printableLine.aIndex}
                            {printableLine.aIndex}
                        {/if}
                    </td>
                    <td>
                        {#if printableLine.bIndex}
                            {printableLine.bIndex}
                        {/if}
                        
                    </td>
                    <td>
                        {#if printableLine.bLine}
                            {#each printableLine.bLine as co}
                                <span class={"diff"+co.idiff}> {co.value} </span>
                            {/each}
                        {:else}
                            <!-- empty -->
                        {/if}
                    </td>
                </tr>
                {/each}
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

    .diff1{
        background-color:#abf2bc;
    }

    .diff-1{
        background-color:rgba(255,129,130,0.4);
    }
</style>