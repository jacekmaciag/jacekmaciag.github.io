<style type="text/css" rel="stylesheet">
* {
	font-size: 13px; 
	line-height: 11px; 
	font-family: 'Lucida Console', Courier, monospace;
	cursor: default;
}

.board_inner_container {
	display: inline-block;
	position: relative;
	width: 100%; 
	height: 100%;
}

#board {
	display: inline-block;
	position: relative;
	z-index: 1;
}

.menu_container {
	display: inline;
	position: absolute;
	padding: 20%;
  	width: 50%;
}

#menu {
	display: inline;
	position: absolute;
}

.btn {
	position: relative;
}

.btn_label { 
	position: absolute;
	padding: 3px;
	text-align: center;
	top: 0; 
	right: 0; 
	bottom: 0; 
	left: 0; 
	width: 100%; 
	height: 100%;
	z-index: 2
}

.btn_overlay {
	position: absolute;
	top: 0; 
	right: 0; 
	bottom: 0; 
	left: 0; 
	width: 100%; 
	height: 100%;
	z-index: 3;
}

.modal {
  	display: none;
  	position: fixed;
  	z-index: 4;
  	left: 0;
  	top: 0;
  	width: 100%;
  	height: 100%;
  	overflow: auto;
  	background-color: rgb(0,0,0);
  	background-color: rgba(0,0,0,0.4);
}

.modal_content {
	background-color: #fefefe;
  	margin: 15% auto;
  	padding: 20px;
  	border: 1px solid #888;
  	width: 50%;
}

.close {
	float: right;
}

.score {
	// implement
}

.field {
	opacity: 1;
	width: 3em;
	height: 6ex;
	position: absolute;
	z-index: 2;
}

.score {
	width: 11em;
	height: 2ex;
	padding-top: 20px;
	font-size: 40px;
}

.field.odd:nth-child(odd) {
	background-color: #E8E8E8;
}

.field.even:nth-child(even) {
	background-color: #E8E8E8;
}

.clickable {
	cursor: pointer;
}

.active {
	background-color: #8FBC8F !important;
}

.figure {
	font-size: 40px; 
	padding-top: 20px;
	text-align: center;
    vertical-align: middle;
    z-index: 3;
}

.move {
	background-color: #FAFAD2 !important;
}

.attack {
	background-color: #F08080 !important;
}
</style>

<div id="container">
	<div class="modal">
		<div id="mm_content" class="modal_content">
			<span class="close">&times;</span>
		</div>
	</div>
	<div id="board_container">
		<div id="computer_score" class="score"></div>
		<div class="board_inner_container">
			<div id="board"></div>
			<div class="menu_container">
				<div id="menu"></div>
			</div>
		</div>
		<div id="player_score" class="score"></div>
	</div>
	<div id="footer"></div>
</div>

<script>
const figureSymbols = {
	"whiteKing": "&#9812;",
	"whiteQueen": "&#9813;",
	"whiteRook": "&#9814;",
	"whiteBishop": "&#9815;",
	"whiteKnight": "&#9816;",
	"whitePawn": "&#9817;",
	"blackKing": "&#9818;",
	"blackQueen": "&#9819;",
	"blackRook": "&#9820;",
	"blackBishop": "&#9821;",
	"blackKnight": "&#9822;",
	"blackPawn": "&#9823;"
}

const directionOffset = {
	'N': {x: 0, y: -1},
	'NE': {x: 1, y: -1},
	'E': {x: 1, y: 0},
	'SE': {x: 1, y: 1},
	'S': {x: 0, y: 1},
	'SW': {x: -1, y: 1},
	'W': {x: -1, y: 0},
	'NW': {x: -1, y: -1}
}

const figureMovement = {
	"whiteKing": {pace: [1], direction: null, attack: null, canSkip: false},
	"whiteQueen": {pace: null, direction: null, attack: null, canSkip: false},
	"whiteRook": {pace: null, direction: ['N', 'E', 'S', 'W'], attack: ['N', 'E', 'S', 'W'], canSkip: false},
	"whiteBishop": {pace: null, direction: ['NE', 'SE', 'SW', 'NW'], attack: ['NE', 'SE', 'SW', 'NW'], canSkip: false},
	"whiteKnight": {pace: [2], direction: ['N', 'E', 'S', 'W', 'NE', 'SE', 'SW', 'NW'], attack: ['N', 'E', 'S', 'W', 'NE', 'SE', 'SW', 'NW'], canSkip: true},
	"whitePawn": {pace: [1], direction: ['N', 'NE', 'NW'], attack: ['NE', 'NW'], canSkip: false},
	"blackKing": {pace: [1], direction: null, attack: null, canSkip: false},
	"blackQueen": {pace: null, direction: null, attack: null, canSkip: false},
	"blackRook": {pace: null, direction: ['N', 'E', 'S', 'W'], attack: ['N', 'E', 'S', 'W'], canSkip: false},
	"blackBishop": {pace: null, direction: ['NE', 'SE', 'SW', 'NW'], attack: ['NE', 'SE', 'SW', 'NW'], canSkip: false},
	"blackKnight": {pace: [2], direction: ['N', 'E', 'S', 'W'], attack: ['N', 'E', 'S', 'W'], canSkip: true},
	"blackPawn": {pace: [1], direction: ['S', 'SE', 'SW'], attack: ['SE', 'SW'], canSkip: false}
}

var hasPlayerMoved = false
var isPlayerCheck = false
var hasComputerMoved = false
var isComputerCheck = false
var playerLost = []
var computerLost = []
var selectedField = null
const clearEvent = new Event('clear')
const redraw = new Event('redraw')
const turn = new Event('turn')

var matrix = new Array(8)
for (var i = 0; i < 8; ++i) {
	matrix[i] = new Array(8)
}

class Field {
	constructor(x, y) {
		this.xPos = x
		this.yPos = y
		this.hasFigure = false
		this.isActive = false
	}
}

function createHSeparator() {
    var hSeparator = document.createElement("span")
    hSeparator.innerHTML = "+---" + "+-----".repeat(8) + "+---+"
    var br = document.createElement("br")
    hSeparator.appendChild(br)
    return hSeparator
}

function createVSeparator(mixIn = null) {
    var vSeparator = document.createElement("span")
    if (mixIn) {
        for (var i = 0; i < mixIn.length; ++i) {
        	if (mixIn[i] === null || mixIn[i] === "") {
				mixIn[i] = "\xa0"
        	}
            if (i == 0) {
                vSeparator.innerHTML += `|\xa0${mixIn[i]}\xa0`
            } else if (0 < i && i < 9) {
            	vSeparator.innerHTML += `|\xa0\xa0${mixIn[i]}\xa0\xa0`
            } else {
                vSeparator.innerHTML += `|\xa0${mixIn[i]}\xa0|`
            }
        }
    } else {
        vSeparator.innerHTML = "|\xa0\xa0\xa0" + "|\xa0\xa0\xa0\xa0\xa0".repeat(8) + "|\xa0\xa0\xa0|"
    }
    var br = document.createElement("br")
    vSeparator.appendChild(br)
    return vSeparator
}

function createBtnLine() {
	var line = document.createElement("span")
	line.innerHTML = "+-------------+"
	return line
}

function createBtnMid(text) { 
	var mid = document.createElement("span")
	mid.style.position = "relative"
	mid.innerHTML = "|" + "\xa0".repeat(13) + "|"
	var label = document.createElement("span")
	label.innerHTML = text
	label.classList.add("btn_label")
	mid.appendChild(label)
	return mid
}

function createBtn(text, clickAction) {
	var btn = document.createElement("div")
	btn.appendChild(createBtnLine())
	btn.appendChild(document.createElement("br"))
	btn.appendChild(createBtnMid(text))
	btn.appendChild(document.createElement("br"))
	btn.appendChild(createBtnLine())
	btn.classList.add("btn")
	var overlay = document.createElement("div")
	overlay.classList.add("btn_overlay")
	overlay.classList.add("clickable")
	overlay.addEventListener('click', clickAction())
	btn.appendChild(overlay)
	return btn
}

function createModal(text) {

}

function offsetX(x) {
	var initOffset = 9
	var fieldOffset = x * 10.5
	return initOffset + fieldOffset
}

function offsetY(y) {
	var initOffset = 7
	var fieldOffset = y * 10.8
	return initOffset + fieldOffset
}

function moveDirection(fromX, fromY, toX, toY) {
	var xDiff = fromX - toX
	var yDiff = fromY - toY
	console.log(xDiff)
	console.log(yDiff)
	var evenFieldAmt = Math.abs(xDiff) == Math.abs(yDiff)
	console.log(evenFieldAmt)
	if (xDiff == 0 && yDiff > 0) {
		return 'N'
	} else if (xDiff < 0 && yDiff > 0 && evenFieldAmt) {
		return 'NE'
	} else if (xDiff < 0 && yDiff == 0) {
		return 'E'
	} else if (xDiff < 0 && yDiff < 0 && evenFieldAmt) {
		return 'SE'
	} else if (xDiff == 0 && yDiff < 0) {
		return 'S'
	} else if (xDiff > 0 && yDiff < 0 && evenFieldAmt) {
		return 'SW'
	} else if (xDiff > 0 && yDiff == 0) {
		return 'W'
	} else if (xDiff > 0 && yDiff > 0 && evenFieldAmt) {
		return 'NW'
	}
}

function movePace(fromX, fromY, toX, toY, direction) {
	var xDiff = fromX - toX
	var yDiff = fromY - toY
	switch(direction) {
		case 'N':
		case 'S':
		pace = Math.abs(yDiff)
		break;
		case 'NE':
		case 'SE':
		case 'SW':
		case 'NW':
		pace = Math.max(Math.abs(yDiff), Math.abs(xDiff))
		break;
		case 'E':
		case 'W':
		pace = Math.abs(xDiff)
		break;
	}
	return pace
}

function checkFieldOccupied(x, y) {
	return matrix[x][y]
}

function getPossibleMoves(posX, posY) {
	var figure = matrix[posX][posY]
	var isKnight = figure.match(/Knight$/)
	var isPawn = figure.match(/Pawn$/)
	var movement = figureMovement[figure]
	var pace = movement.pace ? movement.pace[0] : 8
	if (isPawn && (posY == 6 || posY == 1)) { ++pace }
	var directions = movement.direction ? movement.direction : ['N', 'E', 'S', 'W', 'NE', 'SE', 'SW', 'NW']
	var possibleFields = []
	var blockedDirections = []
	for (var i = 0; i < pace; ++i) {
		for (var j = 0; j < directions.length; ++j) {
			var direction = directions[j]
			// handle skip conditions
			if ((blockedDirections.includes(direction))
				|| (isKnight && i != pace - 1)) {
				continue
			}
			var offset = directionOffset[direction]
			if (offset.x < 0) {
				var xOffset = offset.x - i
			} else if (offset.x > 0) {
				var xOffset = offset.x + i
			} else {
				var xOffset = offset.x
			}
			if (offset.y < 0) {
				var yOffset = offset.y - i
			} else if (offset.y > 0) {
				var yOffset = offset.y + i
			} else { 
				var yOffset = offset.y
			}
			if (isKnight) {
				if (offset.x <= 0 && offset.y < 0) {
					++xOffset 
				} else if (offset.x > 0 && offset.y <= 0) {
					++yOffset
				} else if (offset.x >= 0 && offset.y > 0) {
					--xOffset
				} else if (offset.x < 0 && offset.y >= 0) {
					--yOffset
				}
			}
			candidateX = posX + xOffset
			candidateY = posY + yOffset
			// filter board out-of-bounds results
			if ((candidateX < 0 || candidateX > 7) || (candidateY < 0 || candidateY > 7)) {
				continue 
			}
			if ((isPawn && ['N', 'S'].includes(direction) && matrix[candidateX][candidateY])
				|| (isPawn && ['SE', 'SW', 'NE', 'NW'].includes(direction) && !matrix[candidateX][candidateY])) {
				continue
			}
			if (matrix[candidateX][candidateY]) {
				if((figure.match(/^black/) && matrix[candidateX][candidateY].match(/^white/))
					|| figure.match(/^white/) && matrix[candidateX][candidateY].match(/^black/)) {
					possibleFields.push(`${posX + xOffset}_${posY + yOffset}`)
				}
				blockedDirections.push(direction)
			} else {
				possibleFields.push(`${posX + xOffset}_${posY + yOffset}`)
			}
		}
	}
	console.log(possibleFields)
	return possibleFields
}

function highlightPossibleMoves(fields) { 
	document.querySelectorAll('.field').forEach(function(el) {
		if (fields.includes(el.id)) {
			try {
				if (matrix[el.id.match(/^\d/)][el.id.match(/\d$/)].match(/^black/)) {
					el.classList.add('attack')
				}
			} catch {
				el.classList.add('move')
			}
		}
	})
}

function removeHighlightPossibleMoves(fields) { 
	document.querySelectorAll('.move,.attack').forEach(function(el) {
		el.classList.remove('move')
		el.classList.remove('attack')
	})
}

function selectFigure(figureElement) {
	figureElement.classList.add('active')
	selectedField = figureElement.getAttribute('id')	
}

function unselectFigure(figureElement) {
	figureElement.classList.remove('active')
	selectedField = null
}

function switchToFigure(figureElement) { 
	figureElement.dispatchEvent(clearEvent)
	selectFigure(figureElement)
}

function moveTo(x, y) {
	document.dispatchEvent(clearEvent)
	var selectedPos = {x: parseInt(selectedField.match(/^\d/)), y: parseInt(selectedField.match(/\d$/))}
	selectedField = null
	var figureToMove = matrix[selectedPos.x][selectedPos.y]
	if (getPossibleMoves(selectedPos.x, selectedPos.y).includes(`${x}_${y}`)) {
		matrix[selectedPos.x][selectedPos.y] = null
		if (matrix[x][y]) { computerLost.push(matrix[x][y]) } 
		console.log(computerLost)
		matrix[x][y] = figureToMove
		hasPlayerMoved = true
	}
	draw()
}

function checkWin() {
	if (computerLost.contains("blackKing")) { 
		return true 
	} else {
		return false
	}
}

function createField(x, y, figure = null) {
	var field = new Field(x, y)

	var fieldElem = document.createElement("div")
	fieldElem.style.top = `${offsetY(y)}%`
	fieldElem.style.left = `${offsetX(x)}%`
	fieldElem.setAttribute('id', `${x}_${y}`)
	fieldElem.setAttribute('title', `${["A", "B", "C", "D", "E", "F", "G", "H"][x]}${y + 1}`)
	if (x % 2 == 0){
		fieldElem.setAttribute('class', 'field even')
	} else {
		fieldElem.setAttribute('class', 'field odd')
	}
	if (figure) {
		field.hasFigure = true
		var el = document.createElement("div")
		el.innerHTML = figureSymbols[figure]
		el.classList.add('figure')
		if (figure.match(/^white/)){
			el.classList.add('clickable')
			fieldElem.classList.add('user')
		} else if (figure.match(/^black/)) {
			fieldElem.classList.add('computer')
		}
		fieldElem.appendChild(el)
	}

	fieldElem.addEventListener('click', function() {
		var figure = matrix[x][y]
		if (figure && figure.match(/^white/) && selectedField == null) {
			// select figure
			console.log(`Selecting figure ${figure} at ${fieldElem.getAttribute('id')}`)
			selectFigure(fieldElem)
			var possibleFields = getPossibleMoves(x, y)
			highlightPossibleMoves(possibleFields)
		} else if (figure && figure.match(/^white/) && selectedField == fieldElem.getAttribute('id')) {
			// unselect figure
			console.log(`Unselecting figure ${figure} at ${fieldElem.getAttribute('id')}`)
			unselectFigure(fieldElem)
			removeHighlightPossibleMoves()
		} else if (figure && figure.match(/^white/) && selectedField != fieldElem.getAttribute('id')) {
			// switch to another figure
			console.log(`Switching to figure ${figure} at ${fieldElem.getAttribute('id')}`)
			removeHighlightPossibleMoves()
			switchToFigure(fieldElem)
			var possibleFields = getPossibleMoves(x, y)
			highlightPossibleMoves(possibleFields)
		} else if (selectedField != fieldElem.getAttribute('id')) {
			// move figure
			if (selectedField) { moveTo(x, y) }
			if (hasPlayerMoved) { document.dispatchEvent(turn) }
			hasPlayerMoved = false
		} 
	})

	fieldElem.addEventListener('clear', function() {
		document.querySelectorAll('.field').forEach(function(el){
			el.classList.remove('active')
		})
	})
	return fieldElem
}

function createAllFields(figures) {
	var fields = []
	for (var i = 0; i < 8; ++i) {
		for (var j = 0; j < 8; ++j) {
			var field = createField(i, j, figures[i][j])
			fields.push(field)
		}
	}
	return fields
}

function resetMatrix() {
	matrix[0][0] = "blackRook"
	matrix[1][0] = "blackKnight"
	matrix[2][0] = "blackBishop"
	matrix[3][0] = "blackQueen"
	matrix[4][0] = "blackKing"
	matrix[5][0] = "blackBishop"
	matrix[6][0] = "blackKnight"
	matrix[7][0] = "blackRook"
	for (var i = 0; i < 8; ++i) {
		matrix[i][1] = "blackPawn"
	}
	for (var i = 0; i < 8; ++i) {
		matrix[i][6] = "whitePawn"
	}
	matrix[0][7] = "whiteRook"
	matrix[1][7] = "whiteKnight"
	matrix[2][7] = "whiteBishop"
	matrix[3][7] = "whiteQueen"
	matrix[4][7] = "whiteKing"
	matrix[5][7] = "whiteBishop"
	matrix[6][7] = "whiteKnight"
	matrix[7][7] = "whiteRook"
}

function draw() {
	document.querySelectorAll('.field').forEach(function(el){
		el.remove()
	})
	
	var computerContainer = document.getElementById("computer_score")
	computerContainer.innerHTML = playerLost.map(figure => figureSymbols[figure]).join("")
	computerContainer.classList.add('score')
	var playerContainer = document.getElementById("player_score")
	playerContainer.innerHTML = computerLost.map(figure => figureSymbols[figure]).join("")
	playerContainer.classList.add('score')
	
	var field = createAllFields(matrix)
	for (var i = 0; i < field.length; ++i) {
		document.getElementById("board").appendChild(field[i])
	}
}

function drawBoard() {
	var board = document.createElement("div")
	board.appendChild(createHSeparator())
	board.appendChild(createVSeparator([null, "A", "B", "C", "D", "E", "F", "G", "H", null]))
	board.appendChild(createHSeparator())
	for (var i = 1; i < 9; ++i) {
		board.appendChild(createVSeparator())
		board.appendChild(createVSeparator([i].concat(new Array(8).fill(null)).concat([i])))
		board.appendChild(createVSeparator())
		board.appendChild(createHSeparator())
	}
	board.appendChild(createVSeparator([null, "A", "B", "C", "D", "E", "F", "G", "H", null]))
	board.appendChild(createHSeparator())
	return board
}

function getFigureWithFieldId(id) {
	var x = id.match(/^\d/)
	var y = id.match(/\d$/)
	return matrix[x][y]
}

function fieldIdToCoordinates(fieldId) {
	var x = fieldId.match(/^\d/)[0]
	var y = fieldId.match(/\d$/)[0]
	return {x: parseInt(x), y: parseInt(y)}
}

function getRandomValueFrom(array) {
	return array[Math.floor(Math.random() * array.length)]
}

function getFieldsWithMovableFigures(fields) {
	var movableFigures = {}
	for (var field of fields) {
		var fieldId = field.getAttribute('id')
		var fieldCoordinates = fieldIdToCoordinates(fieldId)
		var possibleMoves = getPossibleMoves(fieldCoordinates.x, fieldCoordinates.y)
		if (possibleMoves.length > 0) {
			movableFigures[fieldId] = possibleMoves
		}
	}
	console.log(movableFigures)
	return movableFigures
}

function computerRandomMove() {
	var computerFields = document.querySelectorAll('.computer')
	console.log(computerFields)
	computerFields = getFieldsWithMovableFigures(computerFields)
	console.log(computerFields)
	var fieldId = getRandomValueFrom(Object.keys(computerFields))
	console.log(fieldId)
	var fieldCoordinates = fieldIdToCoordinates(fieldId)
	console.log(fieldCoordinates)
	var possibleMoves = getPossibleMoves(fieldCoordinates.x, fieldCoordinates.y)
	console.log(possibleMoves)
	var moveToField = getRandomValueFrom(possibleMoves)
	console.log(moveToField)
	var moveToFieldCoordinates = fieldIdToCoordinates(moveToField)
	selectFigure(document.getElementById(fieldId))
	moveTo(moveToFieldCoordinates.x, moveToFieldCoordinates.y)
}

function computerMove() {
	computerRandomMove()
}

function reset() {
	resetMatrix()
	draw()
}

function endGame() {

}

// var menuModal = document.getElementById("main_menu")
// var closeMenuBtn = document.getElementById("close_menu")
// var menuBtn = document.getElementById("open_main_menu")
// menuBtn.addEventListener('click', function() {
// 	menuModal.style.display = "block"
// })

// closeMenuBtn.addEventListener('click', function() {
// 	menuModal.style.display = "none"
// })

// window.addEventListener('click', function(event) {
//   if (event.target == menuModal) {
//     menuModal.style.display = "none";
//   }
// })

document.getElementById("board").appendChild(drawBoard())
document.getElementById("menu").appendChild(createBtn("ABOUT", function(){
	console.log("clicked about")
}))
document.getElementById("menu").appendChild(createBtn("RESET", function(){
	console.log("clicked reset")
}))
document.addEventListener('turn', function() {
	computerMove()
})
reset()
//reset
//about
//difficulty
</script>