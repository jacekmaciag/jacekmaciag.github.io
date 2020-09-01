Some md text here 

<div id="board"></div>

<script>
function createHSeparator() {
    hSeparator = document.createElement("span")
    hSeparator.innerHTML = "#===" + "#=====".repeat(8) + "#===#"
    return hSeparator
}

function createVSeparator(mixIn = null) {
    vSeparator = document.createElement("span")
    if (mixIn) {
        for (i = 0; i < mixIn.length; ++i) {
            if (i == 0){
                vSeparator.innerHTML += "|\xa0${mixIn[i]}\xa0"
            } else if (0 < i < 10) {

            } else if (i == 10) {
                vSeparator.innerHTML += "|\xa0${mixIn[i]}\xa0|"
            }
        }
    } else {
        vSeparator.innerHTML = "|\xa0\xa0\xa0" + "|\xa0\xa0\xa0\xa0\xa0".repeat(8) + "|\xa0\xa0\xa0|"
    }

    return vSeparator
}

function drawBoard() {

}

var board = document.createElement("p")
board.appendChild(createHSeparator())
board.appendChild(createVSeparator(["a", "b", "c", "d"]))
board.appendChild(createHSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())
board.appendChild(createHSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())
board.appendChild(createHSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())
board.appendChild(createVSeparator())

document.getElementById("board").appendChild(board)
</script>
