{ useRef, useEffect, useState } from 'react';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { SketchPicker } from 'react-color';
import { Slider } from '@/components/ui/slider';
import { MoveLeft, MoveRight, Undo, Redo, Users } from 'lucide-react';

export default function SmartBoard() {
  const canvasRef = useRef(null);
  const ctxRef = useRef(null);
  const [tool, setTool] = useState('pen');
  const [color, setColor] = useState('#000000');
  const [lineWidth, setLineWidth] = useState(3);
  const [isDrawing, setIsDrawing] = useState(false);
  const [stickyNotes, setStickyNotes] = useState([]);
  const [pages, setPages] = useState([[]]);
  const [currentPage, setCurrentPage] = useState(0);
  const [history, setHistory] = useState([]);
  const [future, setFuture] = useState([]);

  useEffect(() => {
    const canvas = canvasRef.current;
    canvas.width = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
    const ctx = canvas.getContext('2d');
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    ctx.lineWidth = lineWidth;
    ctxRef.current = ctx;
  }, [lineWidth, currentPage]);

  const saveState = () => {
    const canvas = canvasRef.current;
    setHistory([...history, canvas.toDataURL()]);
    setFuture([]);
  };

  const undo = () => {
    if (history.length === 0) return;
    const canvas = canvasRef.current;
    const ctx = ctxRef.current;
    const last = history.pop();
    setFuture([canvas.toDataURL(), ...future]);
    const img = new Image();
    img.src = last;
    img.onload = () => ctx.drawImage(img, 0, 0);
  };

  const redo = () => {
    if (future.length === 0) return;
    const canvas = canvasRef.current;
    const ctx = ctxRef.current;
    const next = future.shift();
    setHistory([...history, canvas.toDataURL()]);
    const img = new Image();
    img.src = next;
    img.onload = () => ctx.drawImage(img, 0, 0);
  };

  const startDrawing = ({ nativeEvent }) => {
    saveState();
    const { offsetX, offsetY } = nativeEvent;
    ctxRef.current.beginPath();
    ctxRef.current.moveTo(offsetX, offsetY);
    setIsDrawing(true);
  };

  const draw = ({ nativeEvent }) => {
    if (!isDrawing) return;
    const { offsetX, offsetY } = nativeEvent;
    if (tool === 'pen' || tool === 'highlighter') {
      ctxRef.current.strokeStyle = color;
      ctxRef.current.globalAlpha = tool === 'highlighter' ? 0.3 : 1;
      ctxRef.current.lineTo(offsetX, offsetY);
      ctxRef.current.stroke();
    } else if (tool === 'eraser') {
      ctxRef.current.clearRect(offsetX - 8, offsetY - 8, 16, 16);
    }
  };

  const stopDrawing = () => {
    ctxRef.current.closePath();
    setIsDrawing(false);
  };

  const addStickyNote = () => {
    setStickyNotes([
      ...stickyNotes,
      { id: Date.now(), text: 'New Note', x: 100, y: 100, color: '#fef08a' },
    ]);
  };

  const exportBoard = () => {
    const canvas = canvasRef.current;
    const image = canvas.toDataURL('image/png');
    const link = document.createElement('a');
    link.href = image;
    link.download = 'smart-board.png';
    link.click();
  };

  const nextPage = () => {
    if (currentPage + 1 >= pages.length) {
      setPages([...pages, []]);
    }
    setCurrentPage(currentPage + 1);
  };

  const prevPage = () => {
    if (currentPage > 0) setCurrentPage(currentPage - 1);
  };

  return (
    <div className="w-full h-screen bg-gradient-to-br from-slate-100 to-slate-200 p-6">
      <div className="flex flex-wrap gap-4 mb-4 items-center">
        <Button variant={tool === 'pen' ? 'default' : 'outline'} onClick={() => setTool('pen')}>Pen</Button>
        <Button variant={tool === 'highlighter' ? 'default' : 'outline'} onClick={() => setTool('highlighter')}>Highlighter</Button>
        <Button variant={tool === 'eraser' ? 'default' : 'outline'} onClick={() => setTool('eraser')}>Eraser</Button>
        <Button onClick={addStickyNote}>+ Sticky Note</Button>
        <Button onClick={exportBoard}>Export</Button>
        <Button onClick={undo}><Undo className="w-4 h-4 mr-1" />Undo</Button>
        <Button onClick={redo}><Redo className="w-4 h-4 mr-1" />Redo</Button>
        <Button onClick={prevPage}><MoveLeft className="w-4 h-4 mr-1" />Back</Button>
        <Button onClick={nextPage}><MoveRight className="w-4 h-4 mr-1" />Next</Button>
        <Button variant="outline"><Users className="w-4 h-4 mr-1" />Collaborate</Button>

        <div className="ml-4">
          <SketchPicker color={color} onChangeComplete={(col) => setColor(col.hex)} />
        </div>

        <div className="w-48">
          <p className="text-sm font-medium">Brush Size</p>
          <Slider defaultValue={[lineWidth]} max={20} min={1} step={1} onValueChange={(val) => setLineWidth(val[0])} />
        </div>
      </div>

      <div className="relative w-full h-[75vh] bg-white rounded-3xl shadow-2xl overflow-hidden">
        <canvas
          ref={canvasRef}
          className="w-full h-full cursor-crosshair"
          onMouseDown={startDrawing}
          onMouseMove={draw}
          onMouseUp={stopDrawing}
          onMouseLeave={stopDrawing}
        />

        {stickyNotes.map((note) => (
          <Card
            key={note.id}
            className="absolute p-2 w-40 h-40 shadow-md cursor-move"
            style={{ top: note.y, left: note.x, backgroundColor: note.color }}
          >
            <textarea
              className="w-full h-full bg-transparent outline-none resize-none"
              defaultValue={note.text}
            />
          </Card>
        ))}
      </div>
    </div>
  );
}
