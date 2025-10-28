import React, { useState, useMemo } from 'react';
import { LineChart, Line, XAxis, YAxis, Tooltip, CartesianGrid, ResponsiveContainer } from 'recharts';

// Default export React component (ภาษาไทย)
export default function ProvidentSimulator() {
  const [salary, setSalary] = useState(30000);
  const [employeePct, setEmployeePct] = useState(5);
  const [annualReturnPct, setAnnualReturnPct] = useState(2.6);
  const [years, setYears] = useState(20);

  // คำนวณผลลัพธ์โดยใช้การจำลองแบบทบต้นรายปี (เพิ่มยอดสมทบรวมเป็นรายปี แล้วคำนวณดอกเบี้ยทุกสิ้นปี)
  const results = useMemo(() => {
    const employerPct = 2; // นายจ้างสมทบ 2%
    const annualReturn = annualReturnPct / 100; // ผลตอบแทนต่อปี

    let balance = 0;
    const monthlyEmployee = (salary * (employeePct / 100));
    const monthlyEmployer = (salary * (employerPct / 100));

    const annualEmployee = monthlyEmployee * 12;
    const annualEmployer = monthlyEmployer * 12;

    const yearlyData = [];
    let sumEmployeeContrib = 0;
    let sumEmployerContrib = 0;
    let sumInvestmentGain = 0;

    for (let y = 1; y <= years; y++) {
      // สมมติว่าสมทบเป็นรายเดือน แต่รวมเป็นยอดรายปีเพื่อคำนวณดอกเบี้ยรายปี
      // นำยอดสมทบทั้งปีเข้าบัญชี (ถือว่าใส่ในช่วงปีนั้น) แล้วคำนวณดอกเบี้ยเมื่อสิ้นปี
      balance += annualEmployee + annualEmployer;
      sumEmployeeContrib += annualEmployee;
      sumEmployerContrib += annualEmployer;

      // ดอกเบี้ยทบต้นรายปี
      const gain = balance * annualReturn;
      balance += gain;
      sumInvestmentGain += gain;

      yearlyData.push({
        ปี: y,
        ยอดรวม: Math.round(balance),
        สมทบ_ลูกจ้าง: Math.round(sumEmployeeContrib),
        สมทบ_นายจ้าง: Math.round(sumEmployerContrib),
        ผลตอบแทนสะสม: Math.round(sumInvestmentGain),
      });
    }

    return {
      finalBalance: Math.round(balance),
      totalEmployee: Math.round(sumEmployeeContrib),
      totalEmployer: Math.round(sumEmployerContrib),
      totalGain: Math.round(sumInvestmentGain),
      yearlyData,
    };
  }, [salary, employeePct, annualReturnPct, years]);

  return (
    <div className="max-w-4xl mx-auto p-6 bg-white rounded-2xl shadow-md">
      <h1 className="text-2xl font-semibold mb-4">เครื่องมือจำลองกองทุนสำรองเลี้ยงชีพ</h1>
      <p className="text-sm text-gray-600 mb-6">กรอกเงินเดือน เลือกอัตราสมทบ และเลือกผลตอบแทนเพื่อดูยอดสะสมเมื่อครบระยะเวลาที่ต้องการ</p>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
        <label className="flex flex-col">
          <span className="text-sm font-medium">เงินเดือน (บาท)</span>
          <input type="number" min="0" value={salary} onChange={(e) => setSalary(Number(e.target.value))}
                 className="mt-1 p-2 border rounded-lg" />
        </label>

        <label className="flex flex-col">
          <span className="text-sm font-medium">สมทบลูกจ้าง (%)</span>
          <select value={employeePct} onChange={(e) => setEmployeePct(Number(e.target.value))}
                  className="mt-1 p-2 border rounded-lg">
            <option value={2}>2%</option>
            <option value={5}>5%</option>
            <option value={7}>7%</option>
            <option value={10}>10%</option>
          </select>
        </label>

        <label className="flex flex-col">
          <span className="text-sm font-medium">ผลตอบแทนกองทุน (ต่อปี)</span>
          <select value={annualReturnPct} onChange={(e) => setAnnualReturnPct(Number(e.target.value))}
                  className="mt-1 p-2 border rounded-lg">
            <option value={1.8}>1.80%</option>
            <option value={2.6}>2.60%</option>
            <option value={3.3}>3.30%</option>
            <option value={4.03}>4.03%</option>
          </select>
        </label>

        <label className="flex flex-col">
          <span className="text-sm font-medium">ระยะเวลาจำลอง (ปี): {years}</span>
          <input type="range" min="1" max="40" value={years} onChange={(e) => setYears(Number(e.target.value))}
                 className="mt-3" />
        </label>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
        <div className="p-4 bg-gray-50 rounded-lg">
          <div className="text-xs text-gray-500">ยอดรวมเมื่อครบระยะเวลา</div>
          <div className="text-xl font-semibold">{results.finalBalance.toLocaleString()} ฿</div>
        </div>

        <div className="p-4 bg-gray-50 rounded-lg">
          <div className="text-xs text-gray-500">รวมสมทบลูกจ้าง</div>
          <div className="text-lg font-medium">{results.totalEmployee.toLocaleString()} ฿</div>
        </div>

        <div className="p-4 bg-gray-50 rounded-lg">
          <div className="text-xs text-gray-500">รวมสมทบนายจ้าง</div>
          <div className="text-lg font-medium">{results.totalEmployer.toLocaleString()} ฿</div>
        </div>
      </div>

      <div className="mb-6">
        <h2 className="text-lg font-medium mb-2">กราฟการเติบโต (ยอดรวมต่อปี)</h2>
        <div style={{ width: '100%', height: 320 }} className="bg-white rounded-lg p-3 shadow-sm">
          <ResponsiveContainer>
            <LineChart data={results.yearlyData}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="ปี" />
              <YAxis />
              <Tooltip formatter={(value) => `${Number(value).toLocaleString()} ฿`} />
              <Line type="monotone" dataKey="ยอดรวม" stroke="#1f2937" strokeWidth={2} dot={false} />
            </LineChart>
          </ResponsiveContainer>
        </div>
      </div>

      <div className="mb-6">
        <h2 className="text-lg font-medium mb-2">สรุป (ตัวเลขละเอียด)</h2>
        <table className="w-full text-left border-collapse">
          <thead>
            <tr className="text-sm text-gray-600">
              <th className="p-2">ปี</th>
              <th className="p-2">ยอดรวม (฿)</th>
              <th className="p-2">สมทบลูกจ้าง (฿)</th>
              <th className="p-2">สมทบนายจ้าง (฿)</th>
              <th className="p-2">ผลตอบแทนสะสม (฿)</th>
            </tr>
          </thead>
          <tbody>
            {results.yearlyData.map((r) => (
              <tr key={r.ปี} className="border-t">
                <td className="p-2">{r.ปี}</td>
                <td className="p-2">{r.ยอดรวม.toLocaleString()}</td>
                <td className="p-2">{r.สมทบ_ลูกจ้าง.toLocaleString()}</td>
                <td className="p-2">{r.สมทบ_นายจ้าง.toLocaleString()}</td>
                <td className="p-2">{r.ผลตอบแทนสะสม.toLocaleString()}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      <div className="text-sm text-gray-500">
        หมายเหตุ: การจำลองนี้คำนวณจากสมมติฐานว่าเงินเดือนคงที่ตลอดระยะเวลา และสมทบทุกเดือนในอัตราที่ระบุ ผลลัพธ์เป็นการประมาณการเพื่อใช้ประกอบการตัดสินใจเท่านั้น
      </div>
    </div>
  );
}
