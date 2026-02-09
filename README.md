import React, { useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

// Simple demo authentication (not secure)
const ADMIN_PASSWORD = "admin123";

export default function StockRoomWebsite() {
  const [role, setRole] = useState(null); // null | 'admin' | 'viewer'
  const [password, setPassword] = useState("");
  const [loginError, setLoginError] = useState(false);

  const [items, setItems] = useState([
    { id: 1, name: "Paper", qty: 120, updatedAt: new Date() },
    { id: 2, name: "Ink", qty: 45, updatedAt: new Date() },
  ]);

  const [logs, setLogs] = useState([]);

  const deleteLog = (id) => {
    setLogs((prev) => prev.filter((l) => l.id !== id));
  };

  const clearLogs = () => {
    setLogs([]);
  };

  const [newName, setNewName] = useState("");
  const [newQty, setNewQty] = useState("");

  const loginAdmin = () => {
    if (password === ADMIN_PASSWORD) {
      setRole("admin");
      setLoginError(false);
    } else {
      setLoginError(true);
    }
  };

  const addItem = () => {
    if (!newName) return;
    const qty = Number(newQty) || 0;
    const next = {
      id: Date.now(),
      name: newName,
      qty,
      updatedAt: new Date(),
    };
    setItems([...items, next]);

    // log add
    setLogs((l) => [
      {
        id: Date.now() + Math.random(),
        item: newName,
        diff: qty,
        qty,
        time: new Date(),
      },
      ...l,
    ]);

    setNewName("");
    setNewQty("");
  };

  const changeQty = (id, diff) => {
    setItems((prev) =>
      prev.map((it) => {
        if (it.id !== id) return it;
        const nextQty = it.qty + diff;
        const finalQty = nextQty < 0 ? 0 : nextQty;

        setLogs((l) => [
          {
            id: Date.now() + Math.random(),
            item: it.name,
            diff,
            qty: finalQty,
            time: new Date(),
          },
          ...l,
        ]);

        return { ...it, qty: finalQty, updatedAt: new Date() };
      })
    );
  };

  

  const deleteItem = (id) => {
    setItems((prev) => prev.filter((it) => it.id !== id));
  };

  if (!role) {
    return (
      <div className="min-h-screen flex items-center justify-center p-4">
        <Card className="w-full max-w-md rounded-2xl shadow">
          <CardContent className="p-6 space-y-4">
            <h1 className="text-xl font-semibold">Stock Room System</h1>
            <div className="space-y-2">
              <Button className="w-full" onClick={() => setRole("viewer")}>Enter as Staff</Button>
            </div>
            <div className="pt-4 border-t space-y-2">
              <p className="text-sm">Admin Login</p>
              <Input
                placeholder="Password"
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
              />
              <Button className="w-full" onClick={loginAdmin}>Login as Admin</Button>

              {loginError && (
                <div className="flex items-center gap-2 text-red-600 text-sm bg-red-50 border border-red-200 p-2 rounded-xl">
                  <span className="text-lg">✖</span>
                  <span>Wrong password</span>
                </div>
              )}
            </div>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen p-6 space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-semibold">Stock Items</h1>
        <Button variant="outline" onClick={() => setRole(null)}>Logout</Button>
      </div>

      {role === "admin" && (
        <Card className="rounded-2xl shadow">
          <CardContent className="p-4 space-y-3">
            <h2 className="font-medium">Add Item</h2>
            <Input
              placeholder="Item name"
              value={newName}
              onChange={(e) => setNewName(e.target.value)}
            />
            <Input
              placeholder="Quantity"
              type="number"
              value={newQty}
              onChange={(e) => setNewQty(e.target.value)}
            />
            <Button onClick={addItem}>Add</Button>
          </CardContent>
        </Card>
      )}

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {items.map((it) => (
          <Card key={it.id} className="rounded-2xl shadow">
            <CardContent className="p-4 space-y-2">
              <div className={`text-lg font-medium ${it.qty <= 20 ? "text-orange-500" : ""}`}>
                {it.name}
                {it.qty <= 20 && (
                  <span className="ml-2 text-xs bg-orange-100 text-orange-600 px-2 py-1 rounded-lg">ใกล้หมด</span>
                )}
              </div>
              <div className="text-sm text-muted-foreground">Quantity</div>
              <div className="text-xs text-muted-foreground">
                updated: {new Date(it.updatedAt).toLocaleDateString()} {new Date(it.updatedAt).toLocaleTimeString()}
              </div>

              {role === "admin" ? (
                <div className="space-y-2">
                  <div className="flex items-center gap-2">
                    <div className={`text-xl w-16 text-center ${it.qty <= 20 ? "text-orange-500" : ""}`}>{it.qty}</div>
                    <Input
                      type="number"
                      placeholder="จำนวน"
                      className="w-24"
                      onChange={(e) => (it._temp = Number(e.target.value) || 0)}
                    />
                    <Button
                      className="bg-green-600 hover:bg-green-700 text-white"
                      onClick={() => changeQty(it.id, it._temp || 0)}
                    >
                      เพิ่ม
                    </Button>
                    <Button
                      variant="destructive"
                      onClick={() => changeQty(it.id, -(it._temp || 0))}
                    >
                      ลด
                    </Button>
                  </div>
                  <Button variant="destructive" onClick={() => deleteItem(it.id)}>Delete</Button>
                </div>
              ) : (
                <div className={`text-2xl ${it.qty <= 20 ? "text-orange-500" : ""}`}>{it.qty}</div>
              )}
            </CardContent>
          </Card>
        ))}
      </div>

      {/* Logs */}
      <Card className="rounded-2xl shadow">
        <CardContent className="p-4">
          <div className="flex items-center justify-between mb-3">
            <h2 className="font-medium">History</h2>
            {logs.length > 0 && (
              <Button variant="outline" onClick={clearLogs}>Clear</Button>
            )}
          </div>
          <div className="space-y-2 max-h-60 overflow-auto text-sm">
            {logs.length === 0 && <div className="text-muted-foreground">No activity</div>}
            {logs.map((log) => (
              <div key={log.id} className="flex justify-between items-center border-b pb-1 gap-2">
                <div>
                  {log.item} {log.diff > 0 ? `+${log.diff}` : log.diff}
                </div>
                <div className="flex items-center gap-3">
                  <div className="text-muted-foreground">
                    {new Date(log.time).toLocaleDateString()} {new Date(log.time).toLocaleTimeString()}
                  </div>
                  {role === "admin" && (
                    <Button size="sm" variant="destructive" onClick={() => deleteLog(log.id)}>ลบ</Button>
                  )}
                </div>
              </div>
            ))}
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
