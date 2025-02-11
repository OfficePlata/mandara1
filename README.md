"use client";
import React, { useState, useEffect, useRef } from "react";

function MainComponent() {
  const [title, setTitle] = useState("マンダラシート");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  const [librariesLoaded, setLibrariesLoaded] = useState(false);
  const [cells, setCells] = useState({});
  const [isPreviewing, setIsPreviewing] = useState(false);
  const mandalaSheetRef = useRef(null); // マンダラシートの参照

  useEffect(() => {
    const loadLibraries = async () => {
      if (typeof window === "undefined") return;

      try {
        if (!window.html2canvas) {
          const html2canvasScript = document.createElement("script");
          html2canvasScript.src =
            "https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js";
          html2canvasScript.async = true;
          document.body.appendChild(html2canvasScript);
          await new Promise((resolve, reject) => {
            html2canvasScript.onload = resolve;
            html2canvasScript.onerror = reject;
          });
        }

        if (!window.jspdf) {
          const jsPDFScript = document.createElement("script");
          jsPDFScript.src =
            "https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js";
          jsPDFScript.async = true;
          document.body.appendChild(jsPDFScript);
          await new Promise((resolve, reject) => {
            jsPDFScript.onload = resolve;
            jsPDFScript.onerror = reject;
          });
        }

        setLibrariesLoaded(true);
      } catch (err) {
        console.error("ライブラリ読み込みエラー:", err);
        setError("必要なライブラリの読み込みに失敗しました");
      }
    };

    loadLibraries();
  }, []);

  const themes = [
    { text: "学び", color: "bg-red-100" },
    { text: "価値観", color: "bg-blue-100" },
    { text: "成長", color: "bg-yellow-100" },
    { text: "行動", color: "bg-green-100" },
    { text: "楽しい", color: "bg-orange-100" },
    { text: "現罪", color: "bg-purple-100" },
    { text: "目標", color: "bg-violet-100" },
    { text: "欲望", color: "bg-lime-100" },
    { text: "収支", color: "bg-rose-100" },
  ];
  const labels = [
    { pos: "top-left", label: "H" },
    { pos: "top-center", label: "A" },
    { pos: "top-right", label: "B" },
    { pos: "middle-left", label: "G" },
    { pos: "center", label: "" },
    { pos: "middle-right", label: "C" },
    { pos: "bottom-left", label: "F" },
    { pos: "bottom-center", label: "E" },
    { pos: "bottom-right", label: "D" },
  ];

  const handleCellChange = (themeIndex, pos, value) => {
    const key = `${themeIndex}-${pos}`;
    setCells((prev) => ({ ...prev, [key]: value }));
  };

  const adjustFontSize = (text) => {
    let fontSize = 16;
    if (text.length > 10) {
      fontSize = Math.max(10, 16 - (text.length - 10) * 0.3); // 文字数に応じて縮小
    }
    return fontSize;
  };

  const prepareTextForSaving = (text) => {
    // 改行を挿入して視認性を高める
    return text.replace(/([^\n]{50})/g, "$1\n"); // 50文字ごとに改行
  };

  const handleSaveImage = async () => {
    try {
      setLoading(true);
      setError(null);

      const element = mandalaSheetRef.current; // useRefから要素を取得
      if (!element) {
        throw new Error("保存対象の要素が見つかりません");
      }

      const A3_WIDTH = 4961;
      const A3_HEIGHT = 3508;

      // 現在のフォントサイズを保存
      const originalFontSizes = {};
      element.querySelectorAll("textarea").forEach((textarea) => {
        originalFontSizes[textarea.name] = textarea.style.fontSize;
      });

      // テキストエリアのフォントサイズを調整
      element.querySelectorAll("textarea").forEach((textarea) => {
        textarea.style.fontSize = `${adjustFontSize(textarea.value)}px`;
      });

      const options = {
        scale: 2,
        backgroundColor: "#ffffff",
        useCORS: true,
        allowTaint: true,
        logging: false,
        width: element.offsetWidth,
        height: element.offsetHeight,
      };

      const canvas = await html2canvas(element, options);

      // フォントサイズを元に戻す
      element.querySelectorAll("textarea").forEach((textarea) => {
        textarea.style.fontSize = originalFontSizes[textarea.name] || "16px";
      });

      const finalCanvas = document.createElement("canvas");
      finalCanvas.width = A3_WIDTH;
      finalCanvas.height = A3_HEIGHT;
      const ctx = finalCanvas.getContext("2d");

      ctx.fillStyle = "#ffffff";
      ctx.fillRect(0, 0, A3_WIDTH, A3_HEIGHT);

      const scale = Math.min(
        (A3_WIDTH * 0.95) / canvas.width,
        (A3_HEIGHT * 0.95) / canvas.height
      );

      const x = (A3_WIDTH - canvas.width * scale) / 2;
      const y = (A3_HEIGHT - canvas.height * scale) / 2;

      ctx.drawImage(canvas, x, y, canvas.width * scale, canvas.height * scale);

      const dataUrl = finalCanvas.toDataURL("image/png", 1.0);
      const link = document.createElement("a");
      const timestamp = new Date().toISOString().slice(0, 10);
      link.download = `${title || "マンダラシート"}_${timestamp}.png`;
      link.href = dataUrl;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);

      setSuccess(true);
      setTimeout(() => setSuccess(false), 3000);
    } catch (err) {
      console.error("保存エラー:", err);
      setError("画像の保存に失敗しました。もう一度お試しください。");
    } finally {
      setLoading(false);
    }
  };

  const handleSavePDF = async () => {
    try {
      setLoading(true);
      setError(null);

      const element = mandalaSheetRef.current; // useRefから要素を取得
      if (!element) {
        throw new Error("保存対象の要素が見つかりません");
      }

      const A3_WIDTH = 4961;
      const A3_HEIGHT = 3508;

      // 現在のフォントサイズを保存
      const originalFontSizes = {};
      element.querySelectorAll("textarea").forEach((textarea) => {
        originalFontSizes[textarea.name] = textarea.style.fontSize;
      });

      // テキストエリアのフォントサイズを調整
      element.querySelectorAll("textarea").forEach((textarea) => {
        textarea.style.fontSize = `${adjustFontSize(textarea.value)}px`;
      });

      const options = {
        scale: 2,
        backgroundColor: "#ffffff",
        useCORS: true,
        allowTaint: true,
        logging: false,
        width: element.offsetWidth,
        height: element.offsetHeight,
      };

      const canvas = await html2canvas(element, options);

      // フォントサイズを元に戻す
      element.querySelectorAll("textarea").forEach((textarea) => {
        textarea.style.fontSize = originalFontSizes[textarea.name] || "16px";
      });

      const { jsPDF } = window.jspdf;
      const pdf = new jsPDF({
        orientation: "landscape",
        unit: "mm",
        format: "a3",
      });

      const pdfWidth = pdf.internal.pageSize.getWidth();
      const pdfHeight = pdf.internal.pageSize.getHeight();

      const margin = 10;
      const contentWidth = pdfWidth - margin * 2;
      const contentHeight = pdfHeight - margin * 2;

      const scale =
        Math.min(contentWidth / canvas.width, contentHeight / canvas.height) *
        0.95;

      const x = (pdfWidth - canvas.width * scale) / 2;
      const y = (pdfHeight - canvas.height * scale) / 2;

      pdf.addImage(
        canvas.toDataURL("image/png", 1.0),
        "PNG",
        x,
        y,
        canvas.width * scale,
        canvas.height * scale
      );

      const timestamp = new Date().toISOString().slice(0, 10);
      pdf.save(`${title || "マンダラシート"}_${timestamp}.pdf`);

      setSuccess(true);
      setTimeout(() => setSuccess(false), 3000);
    } catch (err) {
      console.error("PDF保存エラー:", err);
      setError("PDFの保存に失敗しました。もう一度お試しください。");
    } finally {
      setLoading(false);
    }
  };

  const cellContent = (themeIndex, posIndex) => {
    const isCenter = labels[posIndex].pos === "center";
    const cellLabel = labels[posIndex]?.label || "";
    const cellKey = `${themeIndex}-${labels[posIndex].pos}`;
    const [cellValue, setCellValue] = useState(cells[cellKey] || "");
    const fontSize = adjustFontSize(cellValue);

    useEffect(() => {
      setCells((prev) => ({ ...prev, [cellKey]: cellValue })); // cellValue が変更されたら cells を更新
    }, [cellValue, cellKey, setCells]); // 依存配列に setCells を追加

    const handleChange = (e) => {
      setCellValue(e.target.value); // ローカルの state を更新
    };

    return (
      <div
        className={`relative w-full h-full flex items-center justify-center ${
          isPreviewing ? "preview-mode" : ""
        }`}
      >
        {!isCenter && cellLabel && !cellValue && (
          <span className="absolute top-1 left-1 text-xs sm:text-sm md:text-base font-bold text-gray-400">
            {cellLabel}
          </span>
        )}

        {isCenter ? (
          <span className="text-sm sm:text-base md:text-xl font-bold text-center whitespace-pre-wrap">
            {themes[themeIndex].text}
          </span>
        ) : (
          <textarea
            name={`cell-${cellKey}`}
            value={cellValue}
            onChange={handleChange}
            className="w-full h-full bg-transparent text-center focus:outline-none resize-none"
            style={{
              fontSize: `${fontSize}px`, // 動的にフォントサイズを適用
              lineHeight: "1.2",
              padding: "8px",
              whiteSpace: "pre-wrap",
              wordBreak: "break-all",
              border: "none",
              overflow: "hidden",
            }}
          />
        )}
      </div>
    );
  };

  return (
    <div className="font-roboto flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
      <div className="w-full max-w-5xl">
        <div className="flex flex-col gap-4 mt-8">
          <div className="flex items-center gap-4">
            <h1 className="text-2xl md:text-3xl font-bold">{title}</h1>
            <button
              onClick={() => {
                if (
                  window.confirm("入力した内容をすべて削除してよろしいですか？")
                ) {
                  setCells({});
                  setTitle("マンダラシート");
                }
              }}
              className="bg-red-500 hover:bg-red-600 text-white text-sm px-3 py-1 rounded-lg transition duration-200"
            >
              リセット
            </button>
          </div>
          <div className="flex items-center gap-4 flex-wrap">
            <input
              type="text"
              value={title}
              onChange={(e) => setTitle(e.target.value)}
              placeholder="ファイル名を入力（任意）"
              className="px-4 py-2 border rounded-lg"
            />
            <button
              onClick={handleSaveImage}
              disabled={loading}
              className="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-6 rounded-lg transition duration-200 disabled:opacity-50 flex items-center gap-2"
            >
              {loading ? (
                <span className="flex items-center gap-2">
                  <span className="animate-spin">⏳</span>
                  保存中...
                </span>
              ) : (
                <>
                  <span>💾</span>
                  画像として保存
                </>
              )}
            </button>
            <button
              onClick={handleSavePDF}
              disabled={loading}
              className="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-6 rounded-lg transition duration-200 disabled:opacity-50 flex items-center gap-2"
            >
              {loading ? (
                <span className="flex items-center gap-2">
                  <span className="animate-spin">⏳</span>
                  保存中...
                </span>
              ) : (
                <>
                  <span>📄</span>
                  PDFとして保存
                </>
              )}
            </button>
          </div>
        </div>
        <div
          id="mandala-sheet"
          className="grid grid-cols-3 gap-4 mt-4 bg-white p-8 rounded-lg"
          style={{
            width: "100%",
            maxWidth: "1200px", // A3横サイズに近い幅
            margin: "0 auto",
          }}
          ref={mandalaSheetRef} // マンダラシートの参照を設定
        >
          {themes.map((theme, themeIndex) => (
            <div
              key={themeIndex}
              className={`${theme.color} border-2 rounded-lg overflow-hidden flex items-center justify-center`} // 中央寄せ
              style={{ aspectRatio: "1 / 1" }} // 正方形を維持
            >
              <div className="grid grid-cols-3 grid-rows-3 w-full h-full">
                {labels.map((pos, posIndex) => (
                  <div
                    key={posIndex}
                    className={`border min-h-[80px] md:min-h-[120px] flex items-center justify-center ${theme.color}`}
                  >
                    {cellContent(themeIndex, posIndex)}
                  </div>
                ))}
              </div>
            </div>
          ))}
        </div>

        {error && (
          <div className="mt-4 w-full bg-red-50 border border-red-200 text-red-600 px-4 py-3 rounded-lg">
            <p className="flex items-center gap-2">
              <span>⚠️</span>
              {error}
            </p>
          </div>
        )}

        {success && (
          <div className="mt-4 w-full bg-green-50 border border-green-200 text-green-600 px-4 py-3 rounded-lg">
            <p className="flex items-center gap-2">
              <span>✅</span>
              保存が完了しました
            </p>
          </div>
        )}
      </div>
       <p className="mt-4 text-gray-500">HUMAN CARE DREAM</p>
        <div className="mt-2">
            <span className="inline-block w-4 h-4 bg-red-500 mr-2"></span>
            <span className="mr-4">マンダラシート</span>
        </div>
    </div>
  );
}

export default MainComponent;
