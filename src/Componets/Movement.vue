<script>
const board = document.getElementById("board");
const turnText = document.getElementById("turnText");

let selectedPiece = null;

let currentTurn = "red";

const pieces = [
    {row:0,col:1,color:"red"},
    {row:0,col:3,color:"red"},
    {row:0,col:5,color:"red"},
    {row:0,col:7,color:"red"},

    {row:1,col:0,color:"red"},
    {row:1,col:2,color:"red"},
    {row:1,col:4,color:"red"},
    {row:1,col:6,color:"red"},

    {row:2,col:1,color:"red"},
    {row:2,col:3,color:"red"},
    {row:2,col:5,color:"red"},
    {row:2,col:7,color:"red"},

    {row:5,col:0,color:"black"},
    {row:5,col:2,color:"black"},
    {row:5,col:4,color:"black"},
    {row:5,col:6,color:"black"},

    {row:6,col:1,color:"black"},
    {row:6,col:3,color:"black"},
    {row:6,col:5,color:"black"},
    {row:6,col:7,color:"black"},

    {row:7,col:0,color:"black"},
    {row:7,col:2,color:"black"},
    {row:7,col:4,color:"black"},
    {row:7,col:6,color:"black"}
];

function drawBoard(){

    board.innerHTML = "";

    turnText.innerHTML =
        currentTurn.charAt(0).toUpperCase() +
        currentTurn.slice(1) +
        " Turn";

    for(let row=0; row<8; row++){

        for(let col=0; col<8; col++){

            const square = document.createElement("div");

            square.classList.add("square");

            if((row + col) % 2 === 0){
                square.classList.add("light");
            } else {
                square.classList.add("dark");
            }

            square.dataset.row = row;
            square.dataset.col = col;

            square.addEventListener("click", movePiece);

            const pieceData = pieces.find(
                p => p.row === row && p.col === col
            );

            if(pieceData){

                const piece = document.createElement("div");

                piece.classList.add("piece");
                piece.classList.add(pieceData.color);

                if(selectedPiece === pieceData){
                    piece.classList.add("selected");
                }

                piece.addEventListener("click",(e)=>{

                    e.stopPropagation();

                    // only select your turn
                    if(pieceData.color !== currentTurn){
                        return;
                    }

                    selectedPiece = pieceData;

                    drawBoard();
                });

                square.appendChild(piece);
            }

            board.appendChild(square);
        }
    }
}

function switchTurn(){

    if(currentTurn === "red"){
        currentTurn = "black";
    } else {
        currentTurn = "red";
    }
}

function movePiece(e){

    if(!selectedPiece) return;

    const newRow = parseInt(e.currentTarget.dataset.row);
    const newCol = parseInt(e.currentTarget.dataset.col);

    const occupied = pieces.find(
        p => p.row === newRow && p.col === newCol
    );

    if(occupied) return;

    let rowDiff = newRow - selectedPiece.row;
    let colDiff = newCol - selectedPiece.col;

    let moved = false;

    // RED MOVEMENT
    if(selectedPiece.color === "red"){

        // normal move
        if(rowDiff === 1 && Math.abs(colDiff) === 1){

            selectedPiece.row = newRow;
            selectedPiece.col = newCol;

            moved = true;
        }

        // eating move
        else if(rowDiff === 2 && Math.abs(colDiff) === 2){

            const middleRow = selectedPiece.row + 1;
            const middleCol = selectedPiece.col + (colDiff / 2);

            const enemy = pieces.find(
                p =>
                p.row === middleRow &&
                p.col === middleCol &&
                p.color === "black"
            );

            if(enemy){

                pieces.splice(pieces.indexOf(enemy),1);

                selectedPiece.row = newRow;
                selectedPiece.col = newCol;

                moved = true;
            }
        }
    }

    // BLACK MOVEMENT
    if(selectedPiece.color === "black"){

        // normal move
        if(rowDiff === -1 && Math.abs(colDiff) === 1){

            selectedPiece.row = newRow;
            selectedPiece.col = newCol;

            moved = true;
        }

        // eating move
        else if(rowDiff === -2 && Math.abs(colDiff) === 2){

            const middleRow = selectedPiece.row - 1;
            const middleCol = selectedPiece.col + (colDiff / 2);

            const enemy = pieces.find(
                p =>
                p.row === middleRow &&
                p.col === middleCol &&
                p.color === "red"
            );

            if(enemy){

                pieces.splice(pieces.indexOf(enemy),1);

                selectedPiece.row = newRow;
                selectedPiece.col = newCol;

                moved = true;
            }
        }
    }

    if(moved){
        switchTurn();
    }

    selectedPiece = null;

    drawBoard();
}

drawBoard();
</script>