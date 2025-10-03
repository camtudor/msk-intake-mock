# msk-intake-mock
Patient intake 
import React, { useMemo, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { CheckCircle2, AlertTriangle, Stethoscope, ChevronRight, ChevronLeft, ClipboardList, FileText, ShieldAlert } from "lucide-react";

// Minimal shadcn-like primitives (assume available in environment)
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Label } from "@/components/ui/label";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

// --- Types

type Intake = {
  patientName: string;
  dob: string;
  bodyArea: "neck" | "thoracic" | "low_back" | "shoulder" | "elbow" | "wrist_hand" | "hip" | "knee" | "ankle_foot" | "other" | "";
  chiefComplaint: string;
  onsetType: "sudden" | "gradual" | "";
  onsetDate: string;
  mechanism: string;
  painCurrent: number;
  painBest: number;
  painWorst: number;
  irritOn: number; // minutes to provoke
  irritOff: number; // minutes to ease
  sleepImpact: boolean;
  aggravators: string;
  easers: string;
  workStatus: "full" | "modified" | "off" | "na" | "";
  psfs1: string; psfs1Score: number;
  psfs2: string; psfs2Score: number;
  psfs3: string; psfs3Score: number;
  prevEpisodes: boolean;
  surgeries: string;
  medications: string;
  imaging: string;
  comorbidities: string;
  // red flags
  rf_saddle: boolean;
  rf_bladder: boolean;
  rf_bowel: boolean;
  rf_progressiveWeak: boolean;
  rf_recentTrauma: boolean;
  rf_unableWB: boolean;
  rf_cancerHx: boolean;
  rf_unexplainedWLorNightPain: boolean;
  // goals & prefs
  goals: string;
  prefs: string[]; // exercise|manual|education
};

const initialIntake: Intake = {
  patientName: "",
  dob: "",
  bodyArea: "",
  chiefComplaint: "",
  onsetType: "",
  onsetDate: "",
  mechanism: "",
  painCurrent: 5,
  painBest: 2,
  painWorst: 8,
  irritOn: 5,
  irritOff: 30,
  sleepImpact: false,
  aggravators: "",
  easers: "",
  workStatus: "",
  psfs1: "Tie shoes", psfs1Score: 4,
  psfs2: "Carry shopping", psfs2Score: 3,
  psfs3: "Sleep through night", psfs3Score: 5,
  prevEpisodes: false,
  surgeries: "",
  medications: "",
  imaging: "",
  comorbidities: "",
  rf_saddle: false,
  rf_bladder: false,
  rf_bowel: false,
  rf_progressiveWeak: false,
  rf_recentTrauma: false,
  rf_unableWB: false,
  rf_cancerHx: false,
  rf_unexplainedWLorNightPain: false,
  goals: "",
  prefs: [],
};

// --- Helpers
function classNames(...cn: (string | false | null | undefined)[]) {
  return cn.filter(Boolean).join(" ");
}

function SectionHeader({ icon: Icon, title, subtitle }: { icon: any; title: string; subtitle?: string }) {
  return (
    <div className="flex items-center gap-3 mb-4">
      <div className="p-2 rounded-2xl bg-gray-100"><Icon className="h-5 w-5" /></div>
      <div>
        <h3 className="text-lg font-semibold">{title}</h3>
        {subtitle && <p className="text-sm text-gray-500">{subtitle}</p>}
      </div>
    </div>
  );
}

function StepPill({ index, active, done, label }: { index: number; active: boolean; done: boolean; label: string }) {
  return (
    <div className={classNames(
      "flex items-center gap-2 px-3 py-1 rounded-full text-sm",
      active ? "bg-black text-white" : done ? "bg-emerald-100 text-emerald-800" : "bg-gray-100 text-gray-600"
    )}>
      <span className="inline-flex items-center justify-center w-5 h-5 rounded-full bg-white/20 border border-white/30">
        {done ? <CheckCircle2 className="h-4 w-4"/> : index}
      </span>
      <span>{label}</span>
    </div>
  );
}

// --- Main component
export default function MSKIntakeMock() {
  const [step, setStep] = useState(0);
  const [data, setData] = useState<Intake>(initialIntake);
  const [consented, setConsented] = useState(false);

  const criticalRedFlag = useMemo(() => {
    const ce = data.rf_saddle || data.rf_bladder || data.rf_bowel;
    const prog = data.rf_progressiveWeak;
    const traumaWB = data.rf_recentTrauma && data.rf_unableWB && (data.bodyArea === "hip" || data.bodyArea === "knee" || data.bodyArea === "ankle_foot");
    return ce || prog || traumaWB;
  }, [data]);

  const priorityFlag = useMemo(() => {
    const cancerCombo = data.rf_cancerHx && data.rf_unexplainedWLorNightPain;
    const highIrrit = data.painWorst >= 8 && data.irritOn <= 5 && data.irritOff >= 30;
    return cancerCombo || highIrrit;
  }, [data]);

  const triage: "escalate" | "priority" | "routine" = criticalRedFlag ? "escalate" : priorityFlag ? "priority" : "routine";

  const summary = useMemo(() => {
    const lines: string[] = [];
    const first = `${data.patientName || "Patient"} with ${data.onsetType || ""} onset ${data.bodyArea || "issue"}.`;
    lines.push(first.trim());
    lines.push(`Onset: ${data.onsetType || "?"}, date ${data.onsetDate || "?"}, mechanism: ${data.mechanism || "?"}.`);
    lines.push(`Irritability: provokes in ${data.irritOn}m, eases in ${data.irritOff}m. Sleep impact: ${data.sleepImpact ? "yes" : "no"}.`);
    lines.push(`Aggravators: ${data.aggravators || "n/a"}. Easers: ${data.easers || "n/a"}.`);
    lines.push(`Function: work ${data.workStatus || "?"}. PSFS → ${data.psfs1} ${data.psfs1Score}/10; ${data.psfs2} ${data.psfs2Score}/10; ${data.psfs3} ${data.psfs3Score}/10.`);
    lines.push(`History: prev episodes ${data.prevEpisodes ? "yes" : "no"}; surgeries ${data.surgeries || "-"}; meds ${data.medications || "-"}; imaging ${data.imaging || "-"}; comorbidity ${data.comorbidities || "-"}.`);
    const rfList = [] as string[];
    if (data.rf_saddle) rfList.push("saddle anaesthesia");
    if (data.rf_bladder) rfList.push("bladder dysfunction");
    if (data.rf_bowel) rfList.push("bowel incontinence");
    if (data.rf_progressiveWeak) rfList.push("progressive weakness");
    if (data.rf_recentTrauma) rfList.push("recent trauma");
    if (data.rf_unableWB) rfList.push("unable to weight-bear");
    if (data.rf_cancerHx) rfList.push("cancer history");
    if (data.rf_unexplainedWLorNightPain) rfList.push("unexplained weight loss / night pain");
    lines.push(`Red flags: ${rfList.length ? rfList.join(", ") : "none reported"}.`);
    lines.push(`Goals: ${data.goals || "-"}. Preferences: ${data.prefs.join(", ") || "-"}.`);
    lines.push(`PROMs: NRS current ${data.painCurrent}/10 (best ${data.painBest}, worst ${data.painWorst}).`);
    lines.push(`Triage: ${triage.toUpperCase()} — ${triage === "escalate" ? "critical red flag present" : triage === "priority" ? "high irritability or risk combo" : "no flags of concern"}.`);
    return lines.join(" \n");
  }, [data, triage]);

  const steps = [
    { label: "Consent" },
    { label: "Complaint" },
    { label: "Pain & Irritability" },
    { label: "Flags" },
    { label: "Function" },
    { label: "History" },
    { label: "Goals" },
    { label: "Review" },
  ];

  function NextPrev() {
    return (
      <div className="flex justify-between mt-6">
        <Button variant="secondary" onClick={() => setStep((s) => Math.max(0, s - 1))} disabled={step === 0}>
          <ChevronLeft className="h-4 w-4 mr-1"/> Back
        </Button>
        <Button onClick={() => setStep((s) => Math.min(steps.length - 1, s + 1))}>
          {step === steps.length - 1 ? "Finish" : "Next"} <ChevronRight className="h-4 w-4 ml-1"/>
        </Button>
      </div>
    );
  }

  function Toggle({ checked, onChange, label }: { checked: boolean; onChange: (v: boolean) => void; label: string }) {
    return (
      <label className="flex items-center gap-3 py-1">
        <input type="checkbox" className="h-4 w-4" checked={checked} onChange={(e)=>onChange(e.target.checked)} />
        <span className="text-sm">{label}</span>
      </label>
    );
  }

  const emergency = criticalRedFlag;

  return (
    <div className="min-h-screen bg-gradient-to-b from-white to-gray-50 py-8 px-4 flex items-center justify-center">
      <Card className="w-full max-w-2xl shadow-xl border-0 rounded-2xl">
        <CardHeader className="pb-2">
          <div className="flex items-center justify-between">
            <CardTitle className="text-2xl font-bold flex items-center gap-2"><Stethoscope className="h-6 w-6"/> MSK Intake (Mock)</CardTitle>
            <div className="hidden md:flex gap-2">
              {steps.map((s, i) => (
                <StepPill key={i} index={i+1} active={i===step} done={i<step} label={s.label} />
              ))}
            </div>
          </div>
        </CardHeader>
        <CardContent>
          <AnimatePresence mode="wait">
            <motion.div key={step} initial={{ opacity: 0, y: 6 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -6 }} transition={{ duration: 0.18 }}>
              {step === 0 && (
                <div>
                  <SectionHeader icon={ClipboardList} title="Consent & Details" subtitle="This is a mock; no data leaves your browser."/>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <Label>Patient name</Label>
                      <Input value={data.patientName} onChange={(e)=>setData({...data, patientName: e.target.value})} placeholder="Alex Morgan"/>
                    </div>
                    <div>
                      <Label>Date of birth</Label>
                      <Input type="date" value={data.dob} onChange={(e)=>setData({...data, dob: e.target.value})}/>
                    </div>
                  </div>
                  <div className="mt-4">
                    <Toggle checked={consented} onChange={setConsented} label="I agree to share this information with my clinician. I understand this is not medical advice."/>
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 1 && (
                <div>
                  <SectionHeader icon={FileText} title="Chief complaint" subtitle="Tell us what's going on"/>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <Label>Body area</Label>
                      <Select value={data.bodyArea} onValueChange={(v)=>setData({...data, bodyArea: v as Intake["bodyArea"]})}>
                        <SelectTrigger><SelectValue placeholder="Select area"/></SelectTrigger>
                        <SelectContent>
                          {[
                            "neck","thoracic","low_back","shoulder","elbow","wrist_hand","hip","knee","ankle_foot","other"
                          ].map(x => <SelectItem key={x} value={x}>{x.replace("_"," ")}</SelectItem>)}
                        </SelectContent>
                      </Select>
                    </div>
                    <div>
                      <Label>Onset</Label>
                      <Select value={data.onsetType} onValueChange={(v)=>setData({...data, onsetType: v as Intake["onsetType"]})}>
                        <SelectTrigger><SelectValue placeholder="sudden or gradual"/></SelectTrigger>
                        <SelectContent>
                          <SelectItem value="sudden">sudden</SelectItem>
                          <SelectItem value="gradual">gradual</SelectItem>
                        </SelectContent>
                      </Select>
                    </div>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mt-4">
                    <div>
                      <Label>Onset date</Label>
                      <Input type="date" value={data.onsetDate} onChange={(e)=>setData({...data, onsetDate: e.target.value})}/>
                    </div>
                    <div className="md:col-span-2">
                      <Label>Mechanism / context</Label>
                      <Input value={data.mechanism} onChange={(e)=>setData({...data, mechanism: e.target.value})} placeholder="e.g., increased sitting; awkward lift; no clear cause"/>
                    </div>
                  </div>
                  <div className="mt-4">
                    <Label>In your own words</Label>
                    <Textarea rows={3} value={data.chiefComplaint} onChange={(e)=>setData({...data, chiefComplaint: e.target.value})} placeholder="Describe your symptoms and what brought them on"/>
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 2 && (
                <div>
                  <SectionHeader icon={Stethoscope} title="Pain & Irritability" subtitle="Your current pain and how easily it flares"/>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div>
                      <Label>Current pain (0–10)</Label>
                      <Input type="range" min={0} max={10} value={data.painCurrent} onChange={(e)=>setData({...data, painCurrent: Number(e.target.value)})}/>
                      <div className="text-sm text-gray-600">{data.painCurrent}/10</div>
                    </div>
                    <div>
                      <Label>Best (24h)</Label>
                      <Input type="range" min={0} max={10} value={data.painBest} onChange={(e)=>setData({...data, painBest: Number(e.target.value)})}/>
                      <div className="text-sm text-gray-600">{data.painBest}/10</div>
                    </div>
                    <div>
                      <Label>Worst (24h)</Label>
                      <Input type="range" min={0} max={10} value={data.painWorst} onChange={(e)=>setData({...data, painWorst: Number(e.target.value)})}/>
                      <div className="text-sm text-gray-600">{data.painWorst}/10</div>
                    </div>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mt-4">
                    <div>
                      <Label>Minutes to provoke</Label>
                      <Input type="number" value={data.irritOn} onChange={(e)=>setData({...data, irritOn: Number(e.target.value)})}/>
                    </div>
                    <div>
                      <Label>Minutes to ease</Label>
                      <Input type="number" value={data.irritOff} onChange={(e)=>setData({...data, irritOff: Number(e.target.value)})}/>
                    </div>
                    <div className="flex items-end">
                      <Toggle checked={data.sleepImpact} onChange={(v)=>setData({...data, sleepImpact: v})} label="Sleep is disturbed by symptoms"/>
                    </div>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
                    <div>
                      <Label>Aggravating factors</Label>
                      <Input value={data.aggravators} onChange={(e)=>setData({...data, aggravators: e.target.value})} placeholder="bending, lifting, sitting..."/>
                    </div>
                    <div>
                      <Label>Easing factors</Label>
                      <Input value={data.easers} onChange={(e)=>setData({...data, easers: e.target.value})} placeholder="walking, heat, breaks..."/>
                    </div>
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 3 && (
                <div>
                  <SectionHeader icon={ShieldAlert} title="Safety check" subtitle="Please tick any that apply"/>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-2">
                    {data.bodyArea === "low_back" && (
                      <>
                        <Toggle checked={data.rf_saddle} onChange={(v)=>setData({...data, rf_saddle: v})} label="New saddle numbness"/>
                        <Toggle checked={data.rf_bladder} onChange={(v)=>setData({...data, rf_bladder: v})} label="Difficulty starting or controlling urination"/>
                        <Toggle checked={data.rf_bowel} onChange={(v)=>setData({...data, rf_bowel: v})} label="Loss of bowel control"/>
                      </>
                    )}
                    <Toggle checked={data.rf_progressiveWeak} onChange={(v)=>setData({...data, rf_progressiveWeak: v})} label="Progressive weakness in a limb"/>
                    <Toggle checked={data.rf_recentTrauma} onChange={(v)=>setData({...data, rf_recentTrauma: v})} label="Recent significant trauma"/>
                    <Toggle checked={data.rf_unableWB} onChange={(v)=>setData({...data, rf_unableWB: v})} label="Unable to fully weight-bear (hip/knee/ankle)"/>
                    <Toggle checked={data.rf_cancerHx} onChange={(v)=>setData({...data, rf_cancerHx: v})} label="History of cancer"/>
                    <Toggle checked={data.rf_unexplainedWLorNightPain} onChange={(v)=>setData({...data, rf_unexplainedWLorNightPain: v})} label="Unexplained weight loss or unrelenting night pain"/>
                  </div>

                  {emergency && (
                    <div className="mt-4 p-4 rounded-xl bg-red-50 border border-red-200 text-red-800">
                      <div className="flex items-center gap-2 font-semibold"><AlertTriangle className="h-5 w-5"/> Potential urgent symptoms</div>
                      <p className="text-sm mt-1">This form can’t give medical advice. If you have new saddle numbness, difficulty starting or controlling urination, or loss of bowel control, please seek urgent care (999/111 or A&E). We’ll alert the clinic.</p>
                    </div>
                  )}
                  <NextPrev />
                </div>
              )}

              {step === 4 && (
                <div>
                  <SectionHeader icon={ClipboardList} title="Function & PSFS" subtitle="Work status and three important activities"/>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div>
                      <Label>Work status</Label>
                      <Select value={data.workStatus} onValueChange={(v)=>setData({...data, workStatus: v as Intake["workStatus"]})}>
                        <SelectTrigger><SelectValue placeholder="Select"/></SelectTrigger>
                        <SelectContent>
                          <SelectItem value="full">full</SelectItem>
                          <SelectItem value="modified">modified</SelectItem>
                          <SelectItem value="off">off</SelectItem>
                          <SelectItem value="na">n/a</SelectItem>
                        </SelectContent>
                      </Select>
                    </div>
                    {[1,2,3].map((n)=> (
                      <div key={n} className="md:col-span-1">
                        <Label>PSFS activity {n}</Label>
                        <Input value={(data as any)[`psfs${n}`]} onChange={(e)=>setData({...data, [`psfs${n}`]: e.target.value} as any)} placeholder={n===1?"Tie shoes": n===2?"Carry shopping":"Sleep through night"}/>
                        <Label className="text-xs mt-1">Score (0–10)</Label>
                        <Input type="number" min={0} max={10} value={(data as any)[`psfs${n}Score`]} onChange={(e)=>setData({...data, [`psfs${n}Score`]: Number(e.target.value)} as any)} />
                      </div>
                    ))}
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 5 && (
                <div>
                  <SectionHeader icon={ClipboardList} title="History" subtitle="Relevant past issues, meds, imaging"/>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <Toggle checked={data.prevEpisodes} onChange={(v)=>setData({...data, prevEpisodes: v})} label="Previous similar episodes"/>
                    <div>
                      <Label>Surgeries (relevant)</Label>
                      <Input value={data.surgeries} onChange={(e)=>setData({...data, surgeries: e.target.value})}/>
                    </div>
                    <div>
                      <Label>Medications</Label>
                      <Input value={data.medications} onChange={(e)=>setData({...data, medications: e.target.value})}/>
                    </div>
                    <div>
                      <Label>Imaging (what/when/results)</Label>
                      <Input value={data.imaging} onChange={(e)=>setData({...data, imaging: e.target.value})}/>
                    </div>
                    <div className="md:col-span-2">
                      <Label>Comorbidities</Label>
                      <Input value={data.comorbidities} onChange={(e)=>setData({...data, comorbidities: e.target.value})}/>
                    </div>
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 6 && (
                <div>
                  <SectionHeader icon={ClipboardList} title="Goals & Preferences"/>
                  <div className="grid grid-cols-1 gap-4">
                    <div>
                      <Label>Goals (what would success look like?)</Label>
                      <Textarea rows={3} value={data.goals} onChange={(e)=>setData({...data, goals: e.target.value})}/>
                    </div>
                    <div className="flex flex-wrap gap-3">
                      {["exercise","manual","education"].map(p => (
                        <button key={p} onClick={()=>{
                          const exists = data.prefs.includes(p);
                          setData({...data, prefs: exists ? data.prefs.filter(x=>x!==p) : [...data.prefs, p]});
                        }} className={classNames("px-3 py-1 rounded-full border", data.prefs.includes(p)?"bg-black text-white border-black":"bg-white text-gray-700 border-gray-300")}>{p}</button>
                      ))}
                    </div>
                  </div>
                  <NextPrev />
                </div>
              )}

              {step === 7 && (
                <div>
                  <SectionHeader icon={ClipboardList} title="Review" subtitle="Preview what your clinician will see"/>
                  <div className="rounded-2xl border p-4 bg-gray-50">
                    <pre className="whitespace-pre-wrap text-sm leading-relaxed">{summary}</pre>
                  </div>
                  <div className="mt-4 flex items-center gap-3">
                    <span className={classNames(
                      "inline-flex items-center gap-2 px-3 py-1 rounded-full text-sm",
                      triage === "escalate" ? "bg-red-100 text-red-800" : triage === "priority" ? "bg-amber-100 text-amber-800" : "bg-emerald-100 text-emerald-800"
                    )}>
                      <AlertTriangle className="h-4 w-4"/> Triage: {triage.toUpperCase()}
                    </span>
                    <span className="text-sm text-gray-500">This mock formats a clinician one‑pager without sending data.</span>
                  </div>
                  <CardFooter className="px-0 mt-6">
                    <Button className="w-full md:w-auto"><FileText className="h-4 w-4 mr-2"/> Export PDF (stub)</Button>
                  </CardFooter>
                </div>
              )}
            </motion.div>
          </AnimatePresence>
        </CardContent>
        <CardFooter className="flex justify-between">
          <div className="flex gap-2 md:hidden">
            {steps.map((s, i) => (
              <div key={i} className={classNames("w-2 h-2 rounded-full", i===step?"bg-black":"bg-gray-300")}/>
            ))}
          </div>
          <div className="ml-auto">
            <div className="text-xs text-gray-500">Internal clinic mock • No backend • UK spelling</div>
          </div>
        </CardFooter>
      </Card>
    </div>
  );
}
