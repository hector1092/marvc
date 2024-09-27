import React, { useState } from 'react';
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"

interface MessageFormProps {
  onSendMessage: (to: string, content: string) => Promise<void>;
}

const MessageForm: React.FC<MessageFormProps> = ({ onSendMessage }) => {
  const [to, setTo] = useState('');
  const [content, setContent] = useState('');
  const [isSending, setIsSending] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSending(true);
    try {
      const messageWithFeedbackLink = `${content}\n\nنرجو منك تقييم الكتاب من خلال الرابط التالي: https://example.com/feedback`;
      await onSendMessage(to, messageWithFeedbackLink);
      setTo('');
      setContent('');
    } catch (error) {
      console.error('Error sending message:', error);
    } finally {
      setIsSending(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <Input
        type="text"
        value={to}
        onChange={(e) => setTo(e.target.value)}
        placeholder="رقم الهاتف"
        required
      />
      <Input
        type="text"
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="محتوى الرسالة"
        required
      />
      <Button type="submit" disabled={isSending}>
        {isSending ? 'جاري الإرسال...' : 'إرسال الرسالة'}
      </Button>
    </form>
  );
};

export default MessageForm;
الآن، لنقم بإنشاء وظيفة لتحميل تقرير التقييمات في ملف جديد:
src/utils/reportUtils.ts
import { BookRating } from '@/types/bookTypes';

export const generateRatingsReport = (ratings: BookRating[]): string => {
  let report = "تقرير تقييمات الكتب\n\n";
  
  // ترتيب الكتب حسب التقييم
  const sortedRatings = [...ratings].sort((a, b) => b.rating - a.rating);
  
  sortedRatings.forEach((book, index) => {
    report += `${index + 1}. ${book.bookName}\n`;
    report += `   التقييم: ${book.rating} نجوم\n`;
    report += `   عدد التقييمات: ${book.reviews}\n\n`;
  });
  
  const bestBook = sortedRatings[0];
  report += `أفضل كتاب لهذا الشهر: ${bestBook.bookName} بتقييم ${bestBook.rating} نجوم\n`;
  
  return report;
};

export const downloadRatingsReport = (ratings: BookRating[]) => {
  const report = generateRatingsReport(ratings);
  const blob = new Blob([report], { type: 'text/plain;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = 'تقرير_تقييمات_الكتب.txt';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};
الآن، لنقم بتحديث مكون BookRatings ليشمل زر تحميل التقرير:
src/components/BookRatings.tsx
import React from 'react';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { Button } from "@/components/ui/button";
import { StarIcon, Download } from 'lucide-react';
import { downloadRatingsReport } from '@/utils/reportUtils';

interface BookRating {
  bookName: string;
  rating: number;
  reviews: number;
}

interface BookRatingsProps {
  ratings: BookRating[];
}

const BookRatings: React.FC<BookRatingsProps> = ({ ratings }) => {
  const topRatedBook = ratings.reduce((prev, current) => (prev.rating > current.rating) ? prev : current);

  return (
    <div className="bg-white p-6 rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-4">تقييمات الكتب</h2>
      <div className="mb-6">
        <h3 className="text-xl font-semibold mb-2">أعلى كتاب تقييماً</h3>
        <p>{topRatedBook.bookName} - التقييم: {topRatedBook.rating} نجوم</p>
      </div>
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>اسم الكتاب</TableHead>
            <TableHead>التقييم</TableHead>
            <TableHead>عدد التقييمات</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {ratings.map((book) => (
            <TableRow key={book.bookName}>
              <TableCell>{book.bookName}</TableCell>
              <TableCell>
                <div className="flex">
                  {[1, 2, 3, 4, 5].map((star) => (
                    <StarIcon
                      key={star}
                      className={`w-4 h-4 ${
                        star <= book.rating ? 'text-yellow-400' : 'text-gray-300'
                      }`}
                    />
                  ))}
                </div>
              </TableCell>
              <TableCell>{book.reviews}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
      <div className="mt-4">
        <Button
          onClick={() => downloadRatingsReport(ratings)}
          className="bg-yellow-400 text-white hover:bg-yellow-500"
        >
          <Download className="mr-2" /> تحميل تقرير التقييمات
        </Button>
      </div>
    </div>
  );
};

export default BookRatings;
أخيراً، لنتأكد من أن الأنواع المستخدمة معرفة بشكل صحيح:
src/types/bookTypes.ts
export interface BookRating {
  bookName: string;
  rating: number;
  reviews: number;
}

